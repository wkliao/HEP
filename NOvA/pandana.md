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

### Parallel Read Strategy for Event Selection
#### Data Partitioning
PandAna's read data partitioning pattern divides the event IDs exclusively into
contiguous ranges of event IDs evenly among all processes. The implementation
includes the followings.
1. Find the total number of unique event IDs. This is essentially the length of
   dataset **'/spill/evt.seq'** whose contents are 0, 1, 2, 3, ...., N-1, if
   its length is N.
2. Calculate the partitioning boundaries, so each process is responsible for a
   exclusive and contiguous range of event IDs. If N is not divisible by P, the
   number of processes, then the remainder IDs are assigned to the processes of
   lower ranks.
3. Note this calculation does not require reading the contents of
   **'/spill/evt.seq'**, but only inquires the length of **'/spill/evt.seq'**.
   ```
   my_count = N / nprocs;
   my_start = my_count * rank;
   if (rank < N % nprocs) {
       my_start += rank;
       my_count++;
   }
   else {
       my_start += N % nprocs;
   }
   my_end = my_start + my_count - 1;
   ```
4. Each process is responsible for the range from 'my_start' to 'my_end' includively.

#### Parallel reads
Each group, **G**, in the concatenated file contains dataset 'evt.seq' whose values 
correspond to '/spill/evt.seq' and can be used to find the data element ranges of
all other datasets in the same group, to be read by all processes.
   * Note all datasets in the same group share the number of rows, i.e. the
     size of first dimension.
   * The contents of **'/G/evt.seq'** are monotonically non-decreasing. It is
     possible to have repeated event IDs in consecutive elements.

Parallel reads consist of the following steps.
1. Read **'/G/evt.seq'** and calculate array index ranges for all processes. 
   This can be done in 3 options.
   a. Option 1. Root process reads the entire '/G/evt.seq' and then broadcasts 
      to the remaining processes. All processes use the contents of
      '/G/evt.seq' to calculate their responsible index ranges. 
   b. Option 2. All processes collectively read the whole '/G/evt.seq'. All
      processes calculate their own responsible index ranges.
   c. Option 3. Only root process reads '/G/evt.seq'. Root calculates
      responsible index ranges for all processes, and calls MPI_Scatter to 
      scatter the boundaries of ranges (start and end) to all other processes.
2. Calculate the responsible index ranges by checking the contents of 
   **'/G/evt.seq'** to find the starting and ending indices that point to range 
   of event IDs fall into its responsible range.
   * Two binary searches should be used, one to search for starting index and
     the other for ending index. This avoid sequentially checking the array
     contents.
3. All processes read the requested datasets in group G collectively, using
   the starting and ending indices (hyperslab), one dataset at a time.
   * Note read ranges are not overlapping among all processes.
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
