# Summary of the Split driver

## Basic structure
The split driver saves the metadata and the raw data into two sperate files. The current underlying file drivers for these two parts are sec2 drivers (POSIX driver).

## Further improvements
1) Use the core driver for the metadata:
    * Benefits: By using the core driver for handling the metadata, the metadata can be read from the file with 1 I/O call. Then, the metadata is all in-memory, which can speed up the metadata related operations.

2) Use the MPI driver for the raw data:
    * Benefits: This can allow the collective/independent file writes/reads to be done on raw data.

3) Put metadata into the raw data file:
There are 3 ideas for doing this.
    1. The first idea is to relocate the metadata to a large boundary. But there will be a big gap in the file.
    2. The second idea is to use POSIX extended attributes to save the metadata.
    3. The third idea is to use a table that can add a level of indirection so that the metadata is relocatable. Then, the relocatable metadata is appended to the raw data file.