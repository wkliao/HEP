# Create Auxiliary Datasets to be Used in Data Partitioning When Reading the Concatenated File in Parallel

The concatenated file will be read in parallel by a separate event query
application. The data partitioning used by the query application is to divide
the datasets based on 3-tuple IDs, such as (run ID, subrun ID, evt ID). In
other words, every dataset element is associated with a 3-tuple. The data
partitioning divides a dataset by assigning its elements with the same 3-tuple
to the same MPI process. Each unique 3-tuples can be "mapped" to a unique integer
referred as 'sequence' and the array of such integers is called `evt.key.seq`.
This auxiliary dataset can be used in data partitioning to figure out the range
of sequences to be assigned to an MPI process.

Another approach is to create an auxiliary dataset called `evt.seq.cnt` which
stores the number of dataset elements sharing the same sequence. Both auxiliary
datasets can be constructed if the below assumptions to the input data files
are fulfilled.

## Assumptions on The Input Data Files
1. Each input file contains data of a unique pair of run and sub-run IDs. The
   run and sub-run IDs are used as part of the input file names, as well as
   stored as two datasets named 'run' and 'subrun' in each group. Contents of
   dataset 'run' are all the same as the run ID as shown in the file name. The
   same for 'subrun'.
2. In each input file, all datasets in the same group share the same 1st
   dimension sizes. Dataset size in one group can be different from another
   group. For example, dataset 'cycle', 'daughterVisE', 'daughterVisEBirks',
   etc. in group 'rec.trk.cosmic.tracks.truthYView' share the same first
   dimension size of 18241.
3. In each input file, every group contains a dataset named 'evt' which stores
   the event IDs of corresponding elements of other datasets in the same group.
   The values in `evt` are of type integer and stored in a monotonically
   nondecreasing order. The adjacent values may be the same. The adjacent
   values may not increment by 1.
4. Other datasets can also be used for data partitioning. In this case, the
   tuple can be of size 4, 5, etc. For example, 'subevt' can be used and its
   contents have the same property as 'evt' described in 3.

### Algorithm A: Create `evt.key.seq` dataset for each group
```
G: groups in input files (all input files share the same "schema")
|G|: number of groups

F: A file set containing all input files assigned to this MPI process.
|F|: The number of input files in F
/* Input files in F may not be read in an increasing order of run and sub-run order. */

/* create a new dataset named "evt.key.seq" in each group in the concatenated file, c */
for each group g = 0, 1, ..., |G|-1:
    /* Each process uses the assigned ID range to construct a new dataset
     * 'evt.key.seq' in each group
     */
    key_seq = malloc(|c.g.evt|) /* concatenated size */
    seq_id = 0  /* locally unique sequence ID */
    j = 0
    for each file f = 0, 1, ..., |F|-1:
        num = |f.g.evt[]|  /* number of elements in dataset 'evt' */
        key_seq[j++] = seq_id
        for i=1, ... num-1
            if (f.g.evt[i] > f.g.evt[i-1])
                seq_id++
            key_seq[j] = seq_id
            j++

    /* num_seq_ids is the number of locally unique sequence IDs */
    num_seq_ids = seq_id + 1

    /* local_nelems is the sum of key_seq[] size across all files assigned */
    local_nelems = j

    /* get the starting offset of sequence ID for this process */
    MPI_Exscan(num_seq_ids, off_seq_ids, 1, MPI_HSIZE_T, MPI_SUM, comm);

    /* add prefix scan offset to local sequence IDs */
    for i=0, 1, ..., local_nelems-1
        key_seq[i] += off_seq_ids

    /* now key_seq[] contains globally unique sequence IDs */

    /* write dataset 'evt.key.seq' to group g of output file c: this process
     * writes from starting file offset of my_range_off[g] with an amount of
     * my_range_len[g] */
    H5Dcreate(c, "g/evt.key.seq", |c.g.evt|, HDF5_I64LE)
    H5Dwrite(c, "g/evt.key.seq", my_range_off[g], my_range_len[g], evtseq)
    free(key_seq)
```

### Algorithm B: Create `evt.key.cnt` dataset for each group
```
G: groups in input files (all input files share the same "schema")
|G|: number of groups

F: A file set containing all input files assigned to this MPI process.
|F|: The number of input files in F
/* Input files in F may not be read in an increasing order of run and sub-run order. */

/* create a new dataset named "evt.key.cnt" in each group in the concatenated file, c */
for each group g = 0, 1, ..., |G|-1:
    /* Each process uses the assigned ID range to construct a new dataset
     * 'evt.key.cnt' in each group
     */
    evt_cnt = malloc(|c.g.evt|) /* concatenated size (an upperbound) */
    cnt_id = 0
    for each file f = 0, 1, ..., |F|-1:
        num = |f.g.evt[]|  /* number of elements in dataset 'evt' */
        evt_cnt[cnt_id] = 1
        for i=1, ... num-1
            if (f.g.evt[i] == f.g.evt[i-1])
                evt_cnt[cnt_id]++
            else
                cnt_id++
                evt_cnt[cnt_id] = 1
        cnt_id++  /* move on to next file */

    /* sizeof_cnt is the number of elements in local evt_cnt */
    sizeof_cnt = cnt_id

    /* get the starting offset of cnt dataset for this process */
    MPI_Exscan(sizeof_cnt, off_cnt, 1, MPI_HSIZE_T, MPI_SUM, comm);

    /* get the global sum of sizeof_cnt of all processes */
    MPI_Allreduce(&sizeof_cnt, tot_sizeof_cnt, 1, MPI_HSIZE_T, MPI_SUM, comm);

    /* create a new dataset 'evt.key.cnt' of an upperbound size */
    H5Dcreate(c, "g/evt.key.cnt", |c.g.evt|, HDF5_I64LE)

    /* write dataset 'evt.key.cnt' to group g of output file c: this process
     * writes from starting offset of off_cnt with an amount of sizeof_cnt
     */
    H5Dwrite(c, "g/evt.key.cnt", off_cnt, sizeof_cnt, evt_cnt)

    /* reduce the size of dataset 'evt.key.cnt' to real size */
    H5Dset_extent(c, "g/evt.key.cnt", tot_sizeof_cnt)

    free(evt_cnt)
```
* Note the size of `evt.key.cnt` in each group can be smaller than `evt` and
  other datasets. */

### Algorithm 3.A: Calculate Data Partitioning Ranges using `evtseq`
```
for each group g = 0, 1, ..., |G|-1:
     Each process reads the entire c.g.evtseq
     p = 0
     prev = 0
     sum = 0
     M = ceil(number of unique values in c.g.evtseq / P)
     for i=0, ..., |c.g.evtseq| - 1
         sum++
         if (sum >= M)
             seq = c.g.evtseq[i]
             while (c.g.evtseq[i+1] == seq)
                 sum++
                 i++
             Proc p reads rows: start = prev, count = sum
             if (p == my_rank)
                 break
             prev += sum
             sum = 0
             p++
```

### Algorithm 3.B: Calculate Data Partitioning Ranges using `evtcnt`
```
for each group g = 0, 1, ..., |G|-1:
     Each process reads the entire c.g.evtcnt
     p = 0
     prev = 0
     sum = 0
     M = ceil(|c.g.evtcnt| / P)
     for i=0, ..., |c.g.evtcnt| - 1
         sum += c.g.evtcnt[i]
         if (sum >= M)
             Proc p reads rows: start = prev, count = sum
             if (p == my_rank)
                 break
             prev += sum
             sum = 0
             p++
```

---
# Older versions

### Algorithm 1: Generate disjoint ID ranges for all processes
```
G: groups in input files (all input files share the same "schema")
|G|: number of groups

F: A file set containing all input files assigned to this MPI process.
|F|: The number of input files in F
/* Input files in F may not be read in an increasing order of run and sub-run order. */

long long my_range_len[|G|];  /* size of ID range, aggregated across assigned files */
long long my_range_off[|G|];  /* starting offset ID for this process */

for each group g = 0, 1, ..., |G|-1:
    my_range_len[g] = 0
    my_range_off[g] = 0
    for each file f = 0, 1, ..., |F|-1:
        my_range_len[g] += |f.g.evt[]|  /* number of elements in dataset 'evt' */

/* calculate the starting offsets of ID ranges to be assigned to all processes */
MPI_Exscan(my_range_len, my_range_off, |G|, MPI_OFFSET, MPI_SUM, MPI_COMM_WORLD);

/* Now, each MPI process is assigned an ID range for each group g. The range
 * starts from my_range_off[g] with size my_range_len[g]
 */
```

### Algorithm 2.A: Create `evtseq` dataset for each group
```
/* create a new dataset named "evtseq" in each group in the concatenated file, c */
for each group g = 0, 1, ..., |G|-1:
    /* Each process uses the assigned ID range to construct a new dataset
     * 'evtseq' in each group
     */
    evtseq = malloc(|c.g.evt|)
    off = my_range_off[g]
    j = 0
    for each file f = 0, 1, ..., |F|-1:
        num = |f.g.evt[]|  /* number of elements in dataset 'evt' */
        evtseq[j++] = off
        for i=1, ... num-1
            evtseq[j] = (f.g.evt[i] == f.g.evt[i-1]) ? evtseq[j-1] : evtseq[j-1] + 1
            j++
        off += num

    /* write dataset 'evtseq' to group g of output file c: this process writes
     * from starting file offset of my_range_off[g] with an amount of
     * my_range_len[g] */
    H5Dcreate(c, "g/evtseq", |c.g.evt|, HDF5_I64LE)
    H5Dwrite(c, "g/evtseq", my_range_off[g], my_range_len[g], evtseq)
    free(c.g.evtseq)
```

### Algorithm 2.B: Create `evtcnt` dataset for each group
```
/* create a new dataset named "evtcnt" in each group in the concatenated file, c */
for each group g = 0, 1, ..., |G|-1:
    /* Each process uses the assigned ID range to construct a new dataset
     * 'evtcnt' in each group
     */
    evtcnt[g] = malloc(|c.g.evt|)
    off = my_range_off[g]
    k = -1
    for each file f = 0, 1, ..., |F|-1:
        num = |f.g.evt[]|  /* number of elements in dataset 'evt' */
        evtcnt[g][++k] = 1
        for i=1, ... num-1
            if (f.g.evt[i] == f.g.evt[i-1])
                evtcnt[g][k]++
            else
                evtcnt[g][++k] = 1
        off += num

    my_evtcnt_len[g] = k+1 /* number of elements in evtcnt[g][] */

MPI_Exscan(my_evtcnt_len, my_evtcnt_off, |G|, MPI_OFFSET, MPI_SUM, MPI_COMM_WORLD);
MPI_Allreduce(my_evtcnt_len, evtcnt_len, |G|, MPI_OFFSET, MPI_SUM, MPI_COMM_WORLD);

for each group g = 0, 1, ..., |G|-1:
    H5Dcreate(c, "g/evtcnt", evtcnt_len[g], HDF5_I64LE)

    /* write dataset 'evtcnt' to group g of output file c: this process writes
     * from starting file offset of my_evtcnt_off[g] with an amount of
     * my_evtcnt_len[g] */
    H5Dwrite(c, "g/evtcnt", my_evtcnt_off[g], my_evtcnt_len[g], evtcnt[g])

    free(evtcnt[q])
```
* Note the size of `evtcnt` in each group can be smaller than other datasets. */

### Algorithm 3.A: Calculate Data Partitioning Ranges using `evtseq`
```
for each group g = 0, 1, ..., |G|-1:
     Each process reads the entire c.g.evtseq
     p = 0
     prev = 0
     sum = 0
     M = ceil(number of unique values in c.g.evtseq / P)
     for i=0, ..., |c.g.evtseq| - 1
         sum++
         if (sum >= M)
             seq = c.g.evtseq[i]
             while (c.g.evtseq[i+1] == seq)
                 sum++
                 i++
             Proc p reads rows: start = prev, count = sum
             if (p == my_rank)
                 break
             prev += sum
             sum = 0
             p++
```

### Algorithm 3.B: Calculate Data Partitioning Ranges using `evtcnt`
```
for each group g = 0, 1, ..., |G|-1:
     Each process reads the entire c.g.evtcnt
     p = 0
     prev = 0
     sum = 0
     M = ceil(|c.g.evtcnt| / P)
     for i=0, ..., |c.g.evtcnt| - 1
         sum += c.g.evtcnt[i]
         if (sum >= M)
             Proc p reads rows: start = prev, count = sum
             if (p == my_rank)
                 break
             prev += sum
             sum = 0
             p++
```
