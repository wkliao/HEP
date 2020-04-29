## Study Note of HEP_HPC example program

### HEP_HPC example program:
  + An short program that follows the same way of creating HDF5 data object as the one used by NOvA.
    It is developed to study the metadata layout in the NOvA file, in hope to find some
    tricks too improve I/O performace, such as by adjusting the metaata block size, the
    order of data object creation.
  + Source codes:
    https://bitbucket.org/fnalscdcomputationalscience/hep_hpc/src/master/examples/make_ntuple_file.cc
  + For the ND/FD data HDF5 files, the metadata is found to be spread out in 
    the file interleaved with raw data blocks. This example program was developed by Marc Paterno
    to mimic the generation of NOvA HDF5 files, in hope it can be used to study the data object
    layouts in files when trying various HDF5 parameter settings.
  + The file produced by this example program contains two groups, each has a few datasets.
    * GROUP "table_1"
      + DATASET "a", a 2D int array of size LEN x 1
      + DATASET "arry", a 3D int array of size LEN x 3 x 4
      + DATASET "b", a 2D double array size LEN x 1
      + DATASET "c", a 2D double array size LEN x 1
      + DATASET "d", a 2D int array size LEN x 1
    * GROUP "table_2"
      + DATASET "x", a 2D double array size LEN x 1
      + DATASET "y", a 2D double array size LEN x 1
      + DATASET "z", a 2D int array size LEN x 1
  + 'LEN' is the command-line option.
  + Command '`h5dump -Hp output.h5`' shows the detailed information of data objects stored in a HDF5 file.
  
---

### Compare files generated from program, original ND data files and repacked ND data files
|                | Generated files | Original files | Repacked files |
| :------------  | :------------   | :------------  | :------------  |
| Group object headers | most group object headers of consecutive groups are placed in order |the object headers of consecutive groups are located far from each other and not ordered | the group object headers of consecutive groups are placed in order |
| Example of group object headers | | - rec.trk.bpf.tracks: 1563536 <br> - rec.trk.bpf.tracks.me: 3157760 <br> - rec.trk.bpf.tracks.me.truth: 3091536 | - rec.trk.bpf.tracks: 8765199 <br> - rec.trk.bpf.tracks.me: 8777722 <br> - rec.trk.bpf.tracks.me.truth: 8781937 |
| Order of metadata | | group object headers (not in order) <br> dataset object headers | group 1 object header <br> dataset object headers of group1 <br> group 2 object header <br> dataset object headers of group2 <br> ... <br> object header of the last group <br> dataset object headers of the last group |
  + Consecutive means that the names are ordered so that they will be visited by H5Ovisit one after another.
  
---

### Test with example program
We modified the example program with different settings. And we collected the file offsets of metadata of the output file and the sequence of HDF5 API calls when the output file was generated to better understand the behavior of the example program.
#### Test 1 (10 groups, dataset size = 1000, default metadata block size 2KB)
* File offsets of the metadata of the output file:
  + table_1 object header: 800
  + table_2 object header: 3520
  + table_3 object header: 6728
  + table_4 object header: 9936
  + table_5 object header: 13144
  + table_6 object header: 16352
  + table_7 object header: 19560
  + table_8 object header: 22768
  + table_9 object header: 25976
  + table_10 object header: 712
* Sequence of HDF5 API calls when created the output file:
  + The creation order of groups is the same as the increasing order of the id of group.
  + Odd behavior: Here I observed an odd group "table_10" which is created after all other groups but has the smallest file offset. In the B-tree, there are 2 leaf nodes. Each of them contains 5 groups. And "table_10" is in the first node with "table_1" to "table_4".   
  
#### Test 2 (2 groups, dataset size = 256, default metadata block size 2KB)
* File offsets of Metadata blocks of the output file:
  + root group symbol table: 96
  + B-tree: 136
  + heap: 680
  + symbol table node: 1504
  + symbol table entry 0 (table_1):
    * object header: 800
    * B-tree: 840
    * heap: 1384
    * symbol table node: 2104
      + object header of dataset "a": 1832
      + object header of dataset "b": 2432
      + object header of dataset "c": 2704
      + object header of dataset "d": 2976
      + object header of dataset "arry": 3248
  + symbol table entry 1 (table_2):
    * object header: 3520
    * B-tree: 3560
    * heap: 4104
    * symbol table node: 4496
      + object header of dataset "x": 4224
      + object header of dataset "y": 4824
      + object header of dataset "z": 5096
      + object header of dataset "a": 5368
      + object header of dataset "b": 5640
      + object header of dataset "c": 5912

* Sequence of HDF5 API calls when created the output file:
  + H5Fcreate out_2_256.h5
  + H5Gcreate2 table_1 open_mode
  + H5Dcreate2 a
  + H5Dcreate2 b
  + H5Dcreate2 c
  + H5Dcreate2 d
  + H5Dcreate2 arry
  + H5Gcreate2 table_2 open_mode
  + H5Dcreate2 x
  + H5Dcreate2 y
  + H5Dcreate2 z
  + H5Dcreate2 a
  + H5Dcreate2 b
  + H5Dcreate2 c
  + H5Dwrite (6 times; write datasets in group table_2)
  + H5Dclose (6 times; close datasets in group table_2)
  + H5Gclose (close group 2)
  + H5Dwrite (5 times; write datasets in group table_1)
  + H5Dclose (5 times; close datasets in group table_1)
  + H5Gclose (close group 1)

* Analysis:
  + The object headers of two groups are right after B-tree and the heap of the file. And while the file offsets of the object headers of two groups are far from each other (800, 3520), the object headers of datasets are located between them. (But the B-tree of each dataset seems to be located far from all other metadata.)
  + The file was only created (and opened) once. The program creates the groups and their datasets and writes to them later. And the increasing order of file offsets of metadata is the same as the creation order.

#### Test 3(2 groups, dataset size = 256, default metadata block size 2KB)
* The location of matadata of the output file:
  + group1 object header
  + datasets object headers in group1
  + group2  object header
  + datasets object headers in group2
  + B-trees of datasets in group2
  + raw data in group2
  + B-trees of datasets in group1
  + raw data in group1
* The object headers of groups and datasets are not interleaved by raw data.

