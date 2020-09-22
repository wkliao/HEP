# Analysis of PandAna I/O Pattern
## Terminologies
* A **Var** is a set of particle properties that should be returned after being filtered out using a Cut.
* A **Cut** is a set of conditions of specific particle properties, that will be used to filter out the slices.
* A **Spectrum** is ...

## Example Var and Cut
`energy` is a Var and `kNumuQuality` is a Cut.
PandAna will return the `calE` values of the slices (rows) in table `rec.slc` which satisfy the conditions specified as `kNumuQuality`.

```
energy = Var(tables['rec.slc']['calE']
kNumuQuality = ['rec.energy.numu']['trkccE'] > 0 & ['rec.sel.remid']['pid'] > 0 & \
               ['rec.slc']['nhit'] > 20 & \
               ['rec.slc']['ncotplanes'] > 4 & \
               ['rec.trk.cosmic']['ntracks'] > 0
```

## Parallel Read Workflow in PandAna
1. Every process reads `evt.seq` dataset from `spill` group.
2. The `evt.seq` values are evenly distributed to all processes. Each process finds its range of the values.
3. Traverse over all the requested groups:
    1. Every process reads `evt.seq` dataset from the group.
    2. Every process finds the rows of the requested datasets, that have its responsible `evt.seq` values.
    3. Each individual process independently reads the rows.

## Research Ideas
Below lists a few ideas about how to improve the dataset concatenation
performance considering the I/O pattern in PandAna.

---

### PandAna's Parallel Read Strategy
#### Data Partitioning
PandAna's read data partitioning pattern divides the event IDs exclusively into
contiguous event IDs among all processes. The implementation includes the
followings.
1. Find the number of unique event IDs. This is essentially the length of
   dataset **'/spill/evt.seq'**. Note the contents of **'/spill/evt.seq'** are
   0, 1, 2, 3, ...., N-1, if its length is N.
2. Calculate the Partitioning boundaries, so each process is responsible for a
   exclusive and contiguous range of event IDs. If N is not divisible by P, the
   number of processes, then the remainder IDs are assigned to the processes in
   lower ranks.
3. Note this calculation does not require reading the contents of
   **'/spill/evt.seq'**, but only inquires the length of **'/spill/evt.seq'**.
#### Parallel reads
For each data group, G, in the concatenated file, do the followings:
1. All processes read the entire of dataset **'/G/evt.seq'** (or a single
   process reads the entire **'/G/evt.seq'** and broadcasts it to all
   processes.)
   * Note all datasets in the same group share the number of rows, i.e. the
     size of first dimension.
   * The contents of **'/G/evt.seq'** are monotonically non-decreasing. It is
     possible to have repeated event IDs in consecutive elements.
2. Each process checks the contents of **'/G/evt.seq'** and finds the starting
   index and length of event IDs fall into its responsible range.
   * A binary search should be used, without sequentially checking the array
     contents.
3. All processes read other datasets in group G collectively, using the
   starting indices and lengths of the subarrays, one dataset at a time.
   * Note the lengths can be different among processes.
   * Read ranges are not overlapping among all processes.

#### I/O and Communications
* For each go() call in PandAna, each process only reads 1 chunk from `evt.seq` dataset in `spill` group.
* For each go() call in PandAna, each process only reads 1/P of `evt.seq` dataset in all the requested groups.
* An allgather(1 integer offset) (step 3) and one allgather(P integer counts) (step 4.3) for each of the requested groups.

---

### Parallel Write for Small Datasets
#### Algorithm
* If the aggregated size of a dataset is smaller than P chunks, only a few processes read the dataset from all the input files.
* In the meantime, the other processes read another dataset that is also smaller than P chunks.
* When writing them, all the small dataset writes are serialized.

#### Expected Benefits
* In HDF5's read-modify-write operations, read cost can be significantly reduced by employing few processes only.
* The data transfer time can also be reduced since the data is read from the input files using few processes only.

---

### Chunk Setting in Dataset Concatenation
When reading datasets, PandAna evenly partitions each dataset to all processes based on `evt.seq`.
If a dataset uses a chunk size which is LCM among the `evt.seq` counts, the duplicated decompression cost in parallel read can be minimized.
