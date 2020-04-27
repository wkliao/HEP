## Study Note of HEP_HPC example program

### HEP_HPC example program:
  + An short program that follows the same way of creating HDF5 data object as the one used by NOvA.
    It is developed to study the metadata layout in the NOvA file, in hope to find some
    tricks too improve I/O performace, such as by adjusting the metaata block size, the
    order of data object creation.
  + Source codes:
    https://bitbucket.org/fnalscdcomputationalscience/hep_hpc/src/master/examples/make_ntuple_file.cc
  + For the ND/FD data HDF5 files, the metadata is found to be spread out in 
    the file interleaved with raw data blocks. This example program generates 
    the HDF5 file in the same way. So we want to study this demo code to figure
    out why these files have such a layout.
### Comparision between original ND data files and repacked ND data files
|                | Original files | Repacked files |
| :------------  | :------------  | :------------  |
| Group object headers | the object headers of consecutive groups are located far from each other and not ordered | the group object headers of consecutive groups are placed in order |
| Example of group object headers | - rec.trk.bpf.tracks: 1563536 <br> - rec.trk.bpf.tracks.me: 3157760 <br> - rec.trk.bpf.tracks.me.truth: 3091536 | - rec.trk.bpf.tracks: 8765199 <br> - rec.trk.bpf.tracks.me: 8777722 <br> - rec.trk.bpf.tracks.me.truth: 8781937 |
| Pattern | group object headers (not in order) <br> dataset object headers | group 1 object header <br> dataset object headers of group1 <br> group 2 object header <br> dataset object headers of group2 <br> ... <br> object header of the last group <br> dataset object headers of the last group |
  + Consecutive means that the names are ordered so that they will be visited by H5Ovisit one after another.

### 2 groups, dataset size = 10, default metadata block size 1KB
* File offsets of Metadata blocks:
  + root group symbol table: 96
  + B-tree: 136
  + heap: 680
  + symbol table node: 1504
  + symbol table entry 0 (group 1):
    * object header: 800
    * B-tree: 840
    * heap: 1384
    * symbol table node: 2104
      + object header of symbol 0: 1832
      + object header of symbol 1: 3248
      + object header of symbol 2: 2432
      + object header of symbol 3: 2704
      + object header of symbol 4: 2976
  + symbol table entry 1 (group 2):
    * object header: 3520
    * B-tree: 3560
    * heap: 4104
    * symbol table node: 4496
      + object header of symbol 0: 4224
      + object header of symbol 1: 4824
      + object header of symbol 2: 5096

* Metadata of the output file from "h5dump -Hp":  
GROUP "/" {  
　　GROUP "table_1" {  
　　　　DATASET "a" {  
　　　　DATATYPE  H5T_STD_I32LE  
　　　　DATASPACE  SIMPLE { ( 10, 1 ) / ( H5S_UNLIMITED, 1 ) }  
　　　　STORAGE_LAYOUT {  
　　　　　　CHUNKED ( 262144, 1 )  
　　　　　　SIZE 1094 (0.037:1 COMPRESSION)  
　　　　}  
　　　　FILTERS {  
　　　　　　PREPROCESSING SHUFFLE  
　　　　　　COMPRESSION DEFLATE { LEVEL 6 }  
　　　　}  
　　　　FILLVALUE {  
　　　　　　FILL_TIME H5D_FILL_TIME_IFSET  
　　　　　　VALUE  H5D_FILL_VALUE_DEFAULT  
　　　　}  
　　　　  ALLOCATION_TIME {  
　　　　  H5D_ALLOC_TIME_INCR  
　　　　}  
　　}  
　　　　DATASET "arry" {  
　　　　　　DATATYPE  H5T_STD_I32LE  
　　　　　　DATASPACE  SIMPLE { ( 10, 3, 4 ) / ( H5S_UNLIMITED, 3, 4 ) }  
　　　　　　STORAGE_LAYOUT {  
　　　　　　　　CHUNKED ( 21845, 3, 4 )  
　　　　　　　　SIZE 1486 (0.323:1 COMPRESSION)  
　　　　　　}  
　　　　　　FILTERS {  
　　　　　　　　PREPROCESSING SHUFFLE  
　　　　　　　　COMPRESSION DEFLATE { LEVEL 6 }  
　　　　　　}  
　　　　　　FILLVALUE {  
　　　　　　　　FILL_TIME H5D_FILL_TIME_IFSET  
　　　　　　　　VALUE  H5D_FILL_VALUE_DEFAULT  
　　　　　　}  
　　　　　　ALLOCATION_TIME {  
　　　　　　　　H5D_ALLOC_TIME_INCR  
　　　　　　}  
　　　　}  
　　　　DATASET "b" {  
　　　　　　DATATYPE  H5T_IEEE_F64LE  
　　　　　　DATASPACE  SIMPLE { ( 10, 1 ) / ( H5S_UNLIMITED, 1 ) }  
　　　　　　STORAGE_LAYOUT {  
　　　　　　　　CHUNKED ( 128, 1 )  
　　　　　　　　SIZE 100 (0.800:1 COMPRESSION)  
　　　　　　}  
　　　　　　FILTERS {  
　　　　　　　　COMPRESSION DEFLATE { LEVEL 6 }  
　　　　　　}  
　　　　　　FILLVALUE {  
　　　　　　　　FILL_TIME H5D_FILL_TIME_IFSET  
　　　　　　　　VALUE  H5D_FILL_VALUE_DEFAULT  
　　　　　　}  
　　　　　　ALLOCATION_TIME {  
　　　　　　　　H5D_ALLOC_TIME_INCR  
　　　　　　}  
　　　　}  
　　　　DATASET "c" {  
　　　　　　DATATYPE  H5T_IEEE_F64LE  
　　　　　　DATASPACE  SIMPLE { ( 10, 1 ) / ( H5S_UNLIMITED, 1 ) }  
　　　　　　STORAGE_LAYOUT {  
　　　　　　　　CHUNKED ( 128, 1 )  
　　　　　　　　SIZE 100 (0.800:1 COMPRESSION)  
　　　　　　}  
　　　　　　FILTERS {  
　　　　　　　　COMPRESSION DEFLATE { LEVEL 6 }  
　　　　　　}  
　　　　　　FILLVALUE {  
　　　　　　　　FILL_TIME H5D_FILL_TIME_IFSET  
　　　　　　　　VALUE  H5D_FILL_VALUE_DEFAULT  
　　　　　　}  
　　　　　　ALLOCATION_TIME {  
　　　　　　　　H5D_ALLOC_TIME_INCR  
　　　　　　}  
　　　　}  
　　　　DATASET "d" {  
　　　　　　DATATYPE  H5T_STD_I32LE  
　　　　　　DATASPACE  SIMPLE { ( 10, 1 ) / ( H5S_UNLIMITED, 1 ) }  
　　　　　　STORAGE_LAYOUT {  
　　　　　　　　CHUNKED ( 128, 1 )  
　　　　　　　　SIZE 52 (0.769:1 COMPRESSION)  
　　　　　　}  
　　　　　　FILTERS {  
　　　　　　　　COMPRESSION DEFLATE { LEVEL 6 }  
　　　　　　}  
　　　　　　FILLVALUE {  
　　　　　　　　FILL_TIME H5D_FILL_TIME_IFSET  
　　　　　　　　VALUE  H5D_FILL_VALUE_DEFAULT  
　　　　　　}  
　　　　　　ALLOCATION_TIME {  
　　　　　　　　H5D_ALLOC_TIME_INCR  
　　　　　　}  
　　　　}  
　　}  
　　GROUP "table_2" {  
　　　　DATASET "x" {  
　　　　　　DATATYPE  H5T_IEEE_F64LE  
　　　　　　DATASPACE  SIMPLE { ( 10, 1 ) / ( H5S_UNLIMITED, 1 ) }  
　　　　　　STORAGE_LAYOUT {  
　　　　　　　　CHUNKED ( 128, 1 )  
　　　　　　　　SIZE 99 (0.808:1 COMPRESSION)  
　　　　　　}  
　　　　　　FILTERS {  
　　　　　　　　COMPRESSION DEFLATE { LEVEL 6 }  
　　　　　　}  
　　　　　　FILLVALUE {  
　　　　　　　　FILL_TIME H5D_FILL_TIME_IFSET  
　　　　　　　　VALUE  H5D_FILL_VALUE_DEFAULT  
　　　　　　}  
　　　　　　ALLOCATION_TIME {  
　　　　　　　　H5D_ALLOC_TIME_INCR  
　　　　　　}  
　　　　}  
　　　　DATASET "y" {  
　　　　　　DATATYPE  H5T_IEEE_F64LE  
　　　　　　DATASPACE  SIMPLE { ( 10, 1 ) / ( H5S_UNLIMITED, 1 ) }  
　　　　　　STORAGE_LAYOUT {  
　　　　　　　　CHUNKED ( 128, 1 )  
　　　　　　　　SIZE 100 (0.800:1 COMPRESSION)  
　　　　　　}  
　　　　　　FILTERS {  
　　　　　　　　COMPRESSION DEFLATE { LEVEL 6 }  
　　　　　　}  
　　　　　　FILLVALUE {  
　　　　　　　　FILL_TIME H5D_FILL_TIME_IFSET  
　　　　　　　　VALUE  H5D_FILL_VALUE_DEFAULT  
　　　　　　}  
　　　　　　ALLOCATION_TIME {  
　　　　　　　　H5D_ALLOC_TIME_INCR  
　　　　　　}  
　　　　}  
　　　　DATASET "z" {  
　　　　　　DATATYPE  H5T_STD_I32LE  
　　　　　　DATASPACE  SIMPLE { ( 10, 1 ) / ( H5S_UNLIMITED, 1 ) }  
　　　　　　STORAGE_LAYOUT {  
　　　　　　　　CHUNKED ( 128, 1 )  
　　　　　　　　SIZE 53 (0.755:1 COMPRESSION)  
　　　　　　}  
　　　　　　FILTERS {  
　　　　　　　　COMPRESSION DEFLATE { LEVEL 6 }  
　　　　　　}  
　　　　　　FILLVALUE {  
　　　　　　　　FILL_TIME H5D_FILL_TIME_IFSET  
　　　　　　　　VALUE  H5D_FILL_VALUE_DEFAULT  
　　　　　　}  
　　　　　　ALLOCATION_TIME {  
　　　　　　　　H5D_ALLOC_TIME_INCR  
　　　　　　}  
　　　　}  
　　}  
}  


* Analysis:
The object headers of two groups are right after B-tree and the heap of the file. And while the file offsets of the object headers of two groups are far from each other (800, 3520), the object headers of datasets are located between them. (But the B-tree of each dataset seems to be located far from all other metadata.)

### 2 groups, dataset size = 256, default metadata block size 1KB
* The location of matadata:
  + group1 object header
  + datasets object headers in group1
  + group2  object header
  + datasets object headers in group2
  + B-trees of datasets in group2
  + raw data in group2
  + B-trees of datasets in group1
  + raw data in group1
* The object headers of groups and datasets are not interleaved by raw data.

