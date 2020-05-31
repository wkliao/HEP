# May 28 meeting summary

## Metadata and raw data layouts
* Metadata allocation: 
    1. Small allocation (metadata size is smaller than the block size):
        1. sub-allocate: the rest space of the block >= the metadata size
        2. grab another block & sub-allocate: the rest space of the block < the metadata size
    2. Large allocation (metadata size is larger than the block size):
        The new large piece of metadata will be appended to the previous end of the allocation.
* 2-level allocations:
    1. Register allocation to the file
    2. Sub-allocation (does not reach file allocation level)
* Small data block: Similar to small metadata block, if raw data blocks are tiny, HDF5 will try to get them packed together.
        
## POSIX I/O and MPI I/O
* POSIX I/O: 
    1. Write: The metadata/raw data will accumulate in the buffer until HDF5 gets a non-contiguous write offset and needs to move to another space in the file. Then, the cached data will be written out. 
    2. 2 flags related to caching in H5FDsec2.c: H5FD_FEAT_ACCUMULATE_METADATA and H5FD_FEAT_DATA_SIEVE.
* MPI I/O: 
    1. Write: Because one processor does not know the requirements of data availability of other processors, there is less caching.

## File offsets of the metadata
* The raw data I/O does not influence the metadata cache. The space of metadata is allocated while it is held in the cache. Only for chunked datasets, when B-tree nodes need to split, more metadata space will have to be allocated.
* The calculation time: 
    1. Some new version data structures: The allocation is deferred until the data is evicted from the cache.
    2. Other data structures: The file offset is calculated when the object is created. eg. version 1 B-tree
* Object-level flush call: The flush can force the space allocated for the metadata in the cache.

## Timing results of files with different metadata block size
* File offsets of metadata are expected to be different.
* Why timings are different?

## H5Ovisit
* Data structures of objects:
    1. H5O_info2_t: Data model information structure. The contents are unrelated to the file format.
    2. H5O_native_info_t: Native file format information structure. The contents depend on the actual file format.
* 3 versions of H5Ovisit:
    1. H5Ovisit: get all the information
    2. H5Ovisit2: get native information & have some control over the other information
    3. H5Ovisit3: only get the data model information (can be controlled with a set of flags)

## Action items
* Flush call: Try the function call H5Fflush and H5Dflush to see how file offsets are affected.
* H5Ovisit: Try different versions and settings of H5Ovisit.
