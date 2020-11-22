# Nov 20 meeting summary

## PandAna I/O behavior analysis
* Problem: the non-scalability of PandAna parallel read when using various numbers of processes and Lustre stripe counts
* Possibility of cause: chunk read contention
* Input file: ND 1951 file
    1. Number of datasets to be read: 123
    2. Lustre stripe count: 32
    3. Lustre stripe size: 1 MiB
    4. Total number of chunks in all 123 datasets: 7344
    
| Number of processes | Aggregate number of chunks read by all processes | Number of chunks read by two or more processes | Most shared chunk is read by number of processes |
| :----: | :-----:| :----:| :-----:|
| 128 | 21,060 | 7,228 | 8 |
| 256 | 34,884 | 7,260 | 16 |
| 512 | 62,532 | 7,275 | 31 |
* Different datasets may behave differently.

## Ideas to improve the PandAna I/O performance
* Faster methods for reading "evt.seq"
    1. 3 methods of reading "evt.seq" datasets:
        1. Method 1: Root process reads the entire 'evt.seq' and then broadcasts to the remaining processes. All processes use the contents of 'evt.seq' to calculate their responsible index ranges. 
        2. Method 2: All processes collectively read the whole 'evt.seq'. All processes calculate their own responsible index ranges.
        3. Method 3: Root process reads 'evt.seq' and calculates responsible index ranges for all processes, and calls MPI_Scatter to scatter the boundaries of ranges (start and end) to all other processes.
    2. Method 3 outperforms the other two methods.

* I/O read request aggregation 
    1. HDF5 design: Each process reads the chunks it needs and decompresses the chunk. A chunk may be read by multiple processes. This approach avoids data redistribution after decompression.
    2. Ideas for improvement:
        1. Use MPI-IO to read, instead H5Dread:
            1. Obtain the offset and size of chunks to be read.
            2. Aggregate chunks across all datasets in a group.
            3. Chunk assignment policies:
                1. A chunk is assigned to whichever access the most.
                2. Evenly assign based on rank IDs.
                3. Others.
            4. Decompress chunks concurrently.
            5. Data redistribution: All-to-all communication to send data to the requested processes.
    3. Adjust policies based on different access patterns of the Nova dataset and other new datasets: percentage of reading time, compression factor, etc.
    4. Freedom of adjustment of PandAna read pattern: align evt division with chunk size.

## Action items:
* Timing breakdown: Get the time percentage of file read, decompression, and other time.
* Study the chunk contention of different datasets: Collect the name of the most shared dataset.
* Come up with new chunk assignment policies.
