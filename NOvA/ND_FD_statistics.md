## Statistics of Neutrino Oscillation Experimental Data Collected From Near and Far Detectors

| Input HDF5 files   | ND            | ND2    | FD    |
| -------------      | ------------: | -----: | ----: |
| Number of files    | 165           | 1951   | 6400  |
| Number of groups   | 999           | 999    | 701   |
| Number of datasets | 15,973        | 15,973 |12,931 |
| Number of 1D datasets (2nd dimension size is 1)            | 15,965 | 15,965 | 12,925 |
| Number of 1D non-zero-sized datasets                       |  2,573 | 2,573  | 3,552 |
| Number of 1D zero-sized datasets (1st dimension size is 0) | 13,392 | 13,392 | 9,373 |
| Number of 2D datasets (2nd dimension size is > 1)          | 8 | 8 | 6 |
| Number of 2D non-zero-sized datasets                       | 6 | 6 | 5 |
| Number of 2D zero-sized datasets (1st dimension size is 0) | 2 | 2 | 1 |
| GZIP compression level                                     | 6 | 6 | 6 |
| HDF5 Chunking      | 128-element based | 128-element based | 128-element based |
| Min file size      | 6.4 MiB           |  6.38 MiB         | 33.91 MiB |
| Max file size      | 293 MiB           |  197.07 MiB       | 97.34 MiB |
| Avg file size      | 218.2 MiB         |  151.61 MiB       | 59.74 MiB |
| Total file size    | 36 GiB            |  288.69 GiB       | 373.34 GiB |

| Input HDF5 datasets (before compression)         | ND            | ND2        | FD         |
| :-------------------------------------------     | ------------: | ---------: | ---------: |
| Sum of all 1D datasets                           |    97.96 GiB  | 783.83 GiB | 413.27 GiB |
| Sum of all 2D datasets                           |   903.17 GiB  | 6.93 TiB   |  16.57 TiB |
| Total of all datasets                            | 1,001.13 GiB  | 7.70 TiB   |  16.98 TiB |
| Max /rec.training.slicemaps/slicemap             |   6.13 GiB    | 3.94 GiB   |   7.10 GiB |
| Max /rec.vtx.elastic.fuzzyk.png2d.cvnmaps/cvnmap | 385.76 MiB    | 249.88 MiB |          - |
| Max /rec.vtx.elastic.fuzzyk.png.cvnmaps/cvnmap   | 371.46 MiB    | 239.52 MiB |     53 MiB |
| Max /rec.training.cvnmaps/cvnlabmap              | 208.18 MiB    | 138.32 MiB |    322 MiB |
| Max /rec.training.cvnmaps/cvnmap                 | 208.18 MiB    | 138.32 MiB |    322 MiB |
| Max /rec.training.cvnmaps/cvnobjmap              | 208.18 MiB    | 138.32 MiB |    322 MiB |
| **Input HDF5 datasets (after compression)**      |               |||
| Sum of all 1D datasets                           |  21.40 GiB    | 169.73 GiB |  69.80 GiB |
| Sum of all 2D datasets                           |   2.05 GiB    | 16.95 GiB  |  32.10 GiB |
| Total of all datasets                            |  23.45 GiB    | 186.69 GiB | 101.90 GiB |
| * Compression setting: GZIP level-6, chunk size is 128-element based | | | |
| * If chunk size is 1 MiB based, the compression ratios can be higher  | | | |


---
### Output concatenated file:

> Note that all the empty datasets are defined with CONTIGUOUS layout in the concatenated file.

| Concatenated HDF5 datasets                       | ND            | ND2        | FD            |
| :-------------------------------------------     | ------------: | ---------: | ------------: |
| Sum of all 1D datasets                           |    12.96 GiB  | 102.22 GiB |     42.38 GiB |
| Sum of all 2D datasets                           |     2.07 GiB  | 16.91 GiB  |     35.50 GiB |
| Sum of all datasets                              |    15.03 GiB  | 119.13 GiB |     77.88 GiB |
| File size                                        |    15.10 GiB  | 119.24 GiB |     78.83 GiB |
| Raw data size                                    |    15.02 GiB  | 119.13 GiB |     77.88 GiB |
| Metadata size                                    |    57.73 MiB  | 86.66 MiB  |    815.26 MiB |
| Unaccounted space                                |    11.69 MiB  | 27.75 MiB  |    158.79 MiB |
| * Compression setting: GZIP level-6, chunk size is 1-MiB based | | |

### Dimensions and counts of datasets in ND concatenated file:

| Dimensions | Type size (Bytes) | No. of datasets | Total Size (MiB) | Size each (MiB) |
| ---------: | ----------------: | --------------: | ---------------: | --------------: |
| **1D datasets** |||||
| `(            0,  1)` | 4 | 13,392 |        0 |       0 |
| `(      193,325,  1)` | 4 |     98 |     72.3 |     0.7 |
| `(      410,679,  1)` | 4 |     52 |     81.5 |     1.6 |
| `(      581,462,  1)` | 4 |     48 |    106.5 |     2.2 |
| `(    1,602,951,  1)` | 4 |    149 |    911.1 |     6.1 |
| `(    1,655,668,  1)` | 4 |     17 |    107.4 |     6.3 |
| `(    1,913,329,  1)` | 4 |     22 |    160.6 |     7.3 |
| `(    1,964,636,  1)` | 4 |    149 |  1,116.7 |     7.5 |
| `(    2,252,332,  1)` | 4 |    149 |  1,280.2 |     8.6 |
| `(    2,299,386,  1)` | 4 |     20 |    175.4 |     8.8 |
| `(    2,308,113,  1)` | 4 |    908 |  7,994.7 |     8.8 |
| `(    2,601,039,  1)` | 4 |    161 |  1,597.5 |     9.9 |
| `(    2,951,430,  1)` | 4 |     19 |    213.9 |    11.3 |
| `(    3,058,419,  1)` | 4 |     19 |    221.7 |    11.7 |
| `(    3,220,655,  1)` | 4 |    395 |  4,852.9 |    12.3 |
| `(    3,432,798,  1)` | 4 |    143 |  1,872.6 |    13.1 |
| `(    6,492,802,  1)` | 4 |    168 |  4,161.0 |    24.8 |
| `(   13,843,014,  1)` | 4 |     28 |  1,478.6 |    52.8 |
| `(   26,490,688,  1)` | 4 |      7 |    707.4 |   101.1 |
| `(  649,021,856,  1)` | 4 |     14 | 34,661.5 | 2,475.8 |
| `(1,695,404,032,  1)` | 4 |      7 | 45,272.2 | 6,567.5 |
|**2D datasets** |||||
| `(        0,   16,000)` | H5T_STD_U8LE | 2 |         0 |         0 |
| `(1,655,668,   16,000)` | H5T_STD_U8LE | 3 |  75,790.5 |  25,263.5 |
| `(2,951,430,   16,000)` | H5T_STD_U8LE | 1 |  45,035.3 |  45,035.2 |
| `(3,058,419,   16,000)` | H5T_STD_U8LE | 1 |  46,667.8 |  46,667.8 |
| `(2,308,113,  344,064)` | H5T_STD_U8LE | 1 | 757,349.6 | 757,349.6 |

### Dimensions and counts of datasets in ND concatenated file:

| Dimensions | Type size (Bytes) | No. of datasets | Total Size (MiB) | Size each (MiB) |
| ---------: | ----------------: | --------------: | ---------------: | --------------: |
| **1D datasets** |||||
| `(            0,  1)` | 4 | 13,392 |        0 |       0 |
| `(      193,325,  1)` | 4 |     98 |     72.3 |     0.7 |
| `(      410,679,  1)` | 4 |     52 |     81.5 |     1.6 |
| `(      581,462,  1)` | 4 |     48 |    106.5 |     2.2 |
| `(    1,602,951,  1)` | 4 |    149 |    911.1 |     6.1 |
| `(    1,655,668,  1)` | 4 |     17 |    107.4 |     6.3 |
| `(    1,913,329,  1)` | 4 |     22 |    160.6 |     7.3 |
| `(    1,964,636,  1)` | 4 |    149 |  1,116.7 |     7.5 |
| `(    2,252,332,  1)` | 4 |    149 |  1,280.2 |     8.6 |
| `(    2,299,386,  1)` | 4 |     20 |    175.4 |     8.8 |
| `(    2,308,113,  1)` | 4 |    908 |  7,994.7 |     8.8 |
| `(    2,601,039,  1)` | 4 |    161 |  1,597.5 |     9.9 |
| `(    2,951,430,  1)` | 4 |     19 |    213.9 |    11.3 |
| `(    3,058,419,  1)` | 4 |     19 |    221.7 |    11.7 |
| `(    3,220,655,  1)` | 4 |    395 |  4,852.9 |    12.3 |
| `(    3,432,798,  1)` | 4 |    143 |  1,872.6 |    13.1 |
| `(    6,492,802,  1)` | 4 |    168 |  4,161.0 |    24.8 |
| `(   13,843,014,  1)` | 4 |     28 |  1,478.6 |    52.8 |
| `(   26,490,688,  1)` | 4 |      7 |    707.4 |   101.1 |
| `(  649,021,856,  1)` | 4 |     14 | 34,661.5 | 2,475.8 |
| `(1,695,404,032,  1)` | 4 |      7 | 45,272.2 | 6,567.5 |
|**2D datasets** |||||
| `(        0,   16,000)` | H5T_STD_U8LE | 2 |         0 |         0 |
| `(1,655,668,   16,000)` | H5T_STD_U8LE | 3 |  75,790.5 |  25,263.5 |
| `(2,951,430,   16,000)` | H5T_STD_U8LE | 1 |  45,035.3 |  45,035.2 |
| `(3,058,419,   16,000)` | H5T_STD_U8LE | 1 |  46,667.8 |  46,667.8 |
| `(2,308,113,  344,064)` | H5T_STD_U8LE | 1 | 757,349.6 | 757,349.6 |

### Dimensions and counts of datasets in FD concatenated file:

| Dimensions | Type size (Bytes) | No. of datasets | Total Size (MiB) | Size each (MiB) |
| ---------: | ----------------: | --------------: | ---------------: | --------------: |
| **1D datasets** |||||
| `(            0,  1)` | 4 | 10,317 |         0 |       0 |
| `(      583,054,  1)` | 4 |     94 |     209.1 |     2.2 |
| `(      647,313,  1)` | 4 |     52 |     128.4 |     2.5 |
| `(    3,089,421,  1)` | 4 |    143 |   1,685.3 |    11.8 |
| `(    3,683,454,  1)` | 4 |     20 |     281.0 |    14.1 |
| `(    5,807,319,  1)` | 4 |     22 |     487.4 |    22.2 |
| `(    5,817,822,  1)` | 4 |     16 |     355.1 |    22.2 |
| `(    6,354,268,  1)` | 4 |    155 |   3,757.1 |    24.2 |
| `(    8,215,180,  1)` | 8 |     43 |   2,695.1 |    62.7 |
| `(    8,247,043,  1)` | 4 |    468 |  14,723.3 |    31.5 |
| `(    9,196,353,  1)` | 4 |    120 |   4,209.8 |    35.1 |
| `(   12,272,390,  1)` | 2 |     46 |   1,076.8 |    23.4 |
| `(   18,029,589,  1)` | 4 |    162 |  11,141.9 |    68.8 |
| `(   41,660,235,  1)` | 4 |    143 |  22,725.7 |   158.9 |
| `(   43,374,942,  1)` | 4 |     17 |   2,812.9 |   165.5 |
| `(   46,337,179,  1)` | 4 |    143 |  25,277.0 |   176.8 |
| `(   46,545,987,  1)` | 4 |    908 | 161,223.4 |   177.6 |
| `(   87,267,330,  1)` | 4 |     14 |   4,660.6 |   332.9 |
| `(   93,085,152,  1)` | 4 |      7 |   2,485.6 |   355.1 |
| `(  264,893,646,  1)` | 4 |     28 |  28,293.7 | 1,010.5 |
| `(1,662,482,432,  1)` | 4 |      7 |  44,393.1 | 6,341.9 |
| **2D datasets** |||||
| `(         0,   16,000)` | H5T_STD_U8LE | 1 |            0 |            0 |
| `( 8,215,180,   16,000)` | H5T_STD_U8LE | 1 |    125,353.7 |    125,353.7 |
| `(43,374,942,   16,000)` | H5T_STD_U8LE | 3 |  1,985,547.3 |    661,849.1 |
| `(46,545,987,  344,064)` | H5T_STD_U8LE | 1 | 15,272,902.0 | 15,272,902.0 |

---

* Table Hierarchy
  + We found three types of groups.
    - Tables with 3 IDs, `run`, `subrun`, `evt`. These tables have unique event IDs in `evt` dataset.
    - Tables with 3 IDs, `run`, `subrun`, `evt`, and an index dataset such as `xxx_idx`. These tables have duplicated event IDs in `evt` dataset.
    - Tables with 4 IDs, `run`, `subrun`, `evt`, and `subevt`. These tables also have duplicated event IDs in `evt`.
  + Within each file, only 14 groups have 3 IDs.
  + `spill` has unique `evt` IDs. We need to verify if there are any more tables like `spill`.
    * For ND data, the size of concatenated datasets in group `spill` is 410,679
    * For FD data, the size of concatenated datasets in group `spill` is 647,313   
  + So far, we did not see any files that have the same `run` and `subrun` IDs.
