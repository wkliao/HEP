# Distributed GNN Meeting Summary (August 20, 2020)
This file summarizes the key points made during the meeting with Xiangyang Ju from LBNL. 

## Current Implementation
* Two versions of Parallel GNN implementations: 
  - Bare tensorflow (TF) version 
  - Tensorflow version using Horovod library 
* Better performance using Horovod distributed library
* For bare TF version, graphs were padded so all samples have the same size
* Link to codebase: https://github.com/xju2/heptrkx-gnn-tracking/tree/tf2
  - Bare TF version: https://github.com/xju2/heptrkx-gnn-tracking/blob/tf2/scripts/train_nx_graph_tpu
  - TF + Horovod version: https://github.com/xju2/heptrkx-gnn-tracking/blob/tf2/scripts/hvd_distributed

## Issues and Future Research direction 
* Faced issues of memory footprint issue due to large sample sizes
* Made adjustments by creating a threshhold to exclude the samples with largest number of nodes/edges 
* Link to powerpoint with graphs and results: https://docs.google.com/presentation/d/1SakmdsVgjqWGwyXG1Ur9hyuh_LM64Gcj39cp82sQdcY/edit#slide=id.g8c9e8ec0a2_0_345 

## Publications
* [Track Seeding and Labelling with Embedded-space Graph Neural Networks](https://arxiv.org/pdf/2007.00149.pdf) on Connecting the Dots Workshop (CTD 2020)
