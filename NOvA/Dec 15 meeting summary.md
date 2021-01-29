# Dec 15 meeting summary

## Generalization of HDF5 concatenation program
* Problem Derek has encountered:
    1. Derek uses "run, subrun, cycle, evt" as the key.
    2. Not all groups contain the "cycle" column. (He skipped that table.) The tables contain subevt level data have "cycle" column. 
    3. Every file contains the dataset "cycle," and the values are the same for the same file.
    4. Derek will pull the request to the repo.
* Expand the concatenation program and utility program:
    1. Considering the generalization, which allows users to specify a set of datasets as key. Then the number of columns used to index can be different for different experiments.
    2. The table that contains the unique value can be specified by the user. (eg. "spill")

## Study of the PandAna I/O
* The cause of the non-scalability of PandAna parallel read: The average number of chunks read by each process is not reduced proportionally when the number of processes increases.
* Data parallelism: 
    1. The method with the best performance: 
        1. Read evt.seq method: MPI collective read all evt.seq, decompress, and scatters boundaries.
        2. Read other datasets method: MPI collective read and decompress, all datasets in one group at a time.
        3. This method is not scalable.
        4. Prof. Liao suggested looking for features of python MPI to implement this method in the python program. Marc suggested using the C interface. 
    2. Weak scaling
* Task parallelism:
    1. Group parallelism.
    2. Marc suggested using MPI to distribute the data if the I/O performance improvement is big enough. Prof. Liao mentioned the MPI is doing a similar thing, and if the data amount is large, MPI cannot scale. Marc mentioned it is possible to switch the way of calculation of PandAna to do more calculation within the groups. 

## Action items
* Incorporate Derek's modification
* Make the data parallelism method work with the current PandAna program
* Try other task parallelism method, such as dataset parallelism
