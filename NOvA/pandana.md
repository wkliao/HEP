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
1. Every process reads `evtseq` dataset from `spill` group.
2. The `evtseq` values are evenly distributed to all processes. Each process finds its range of the values.
3. Traverse over all the requested groups:
    1. Every process reads `evtseq` dataset from the group.
    2. Every process finds the rows of the requested datasets, that have its responsible `evtseq` values.
    3. Each individual process independently reads the rows.

## Research Ideas
Below lists a few ideas about how to improve the dataset concatenation
performance considering the I/O pattern in PandAna.

---

### Parallel Read Strategy for PandAna
#### Algorithm
1. Every process reads the length of `evtseq` dataset in `spill` group and calculates how many `evtseq` values should be covered by each process.
2. Every process reads the last `evtseq` value of its own range.
3. All the processes perform allgather() for their last `evtseq` value.
4. Traverse over all the requested groups:
    1. Every process reads 1/P of `evtseq` dataset in a group (or a single process reads the entire `evtseq` and scatter it).
    2. Every process counts how many rows should be covered by each process using the `evtseq` values obtained in step 3. The output of this step is P integers per process.
    3. Allgather() for the P integers (counts). Then, every process calculates which rows it should read from the datasets in this group using the gathered counts.

#### I/O and Communications
* For each go() call in PandAna, each process only reads 1 chunk from `evtseq` dataset in `spill` group.
* For each go() call in PandAna, each process only reads 1/P of `evtseq` dataset in all the requested groups.
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
When reading datasets, PandAna evenly partitions each dataset to all processes based on `evtseq`.
If a dataset uses a chunk size which is LCM among the `evtseq` counts, the duplicated decompression cost in parallel read can be minimized.
