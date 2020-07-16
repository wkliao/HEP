# July 14, 2020 GNN Meeting Summary
This file summarizes the key points made during the meeting

## Feature Training 
Current training samples contain only part of the available data (9 features per hit) Included is the link for the HDF5 generation code for the Dune dataset: https://cdcvs.fnal.gov/redmine/projects/dunetpc/repository/revisions/develop/entry/dune/CVN/art/GCNH5_module.cc


## Different Model Architectures 
Other types of GNN models are being considered; in particular one with graph clustering and graph pooling layers. 

## HDF5 Concatenation
Looking into concatenating all input files into a single larger file. Provided our HDF5 file concatenation tool: https://github.com/NU-CUCIS/ph5concat

## Model Training Timing Table 
|                  Measurement                         Timing       |
| :------------------------------------------: | :----------------: |
| Number of Iterations Per Epoch               | 6266 iterations    |
| Average I/O time over several epochs         | 3211.1835 seconds  |
| Average Computation time over several epochs | 8225.5312 seconds  |
| Average End-to-End time over several epochs  | 11436.7148 seconds |
| Model Accuracy                               | 85% +- 2/3%        |

