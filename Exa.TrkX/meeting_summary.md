# GNN Meeting Summary (July 14, 2020)
This file summarizes the key points made during the meeting

## Feature Training 
Current training samples contain only part of the available data (9 features per hit) Included is the link for the HDF5 generation code for the Dune dataset: https://cdcvs.fnal.gov/redmine/projects/dunetpc/repository/revisions/develop/entry/dune/CVN/art/GCNH5_module.cc


## Different Model Architectures 
Other types of GNN models are being considered; in particular one with graph clustering and graph pooling layers. 

## HDF5 Concatenation
Looking into concatenating all input files into a single larger file. Provided our HDF5 file concatenation tool: https://github.com/NU-CUCIS/ph5concat

## Model Training Timing Table 

|                  Measurement                 |       Timing       |
| :------------------------------------------: | :----------------: |
| Number of Iterations Per Epoch               | 6266 iterations    |
| Average I/O time over several epochs         | 3211.1835 seconds  |
| Average Computation time over several epochs | 8225.5312 seconds  |
| Average End-to-End time over several epochs  | 11436.7148 seconds |
| Model Accuracy                               | 85% +- 2/3%        |

## Areas for Investigation
	* Based on our performance evaluation on a local GPU machine, I/O operations (28% of the total training time) are not negligible. We will further explore this aspect and possibly find a way to reduce the cost. 

	* Concatenating input files could potentially decrease data loading time by saving time for file open/close

	* Scaling up with more GPUs could decrease data loading time by increasing the degree of parallelism with more threads 

## Steps for Studying Performance Behavior
	* Study the effectiveness of the current model and consider further improvements

	* Try combining Pytorch files to decrease file I/O time for data loading during training

	* Track the number of threads used to execute the data loading function __getitem__()

	* Scale up the GNN code on multiple GPUs and/or on a supercomputer and analyze the performance

	* Check if the read/write bandwidth matches the I/O hardware performance peak

	* Study different GNN models with graph pooling layers and/or clustering

