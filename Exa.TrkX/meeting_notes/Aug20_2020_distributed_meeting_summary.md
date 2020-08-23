
# Distributed GNN Meeting Summary (August 20, 2020)
This file summarizes the key points made during the meeting with Xiangyang from Berkeley 

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

## Meeting Attendees
Xiangyang, Jim, Sunwoo, Claire


## Afterthoughts and Action Items
Sunwoo's thoughts:

1. Xiangyang has been studying distributed training as an explorative trial.
For now, he does not have a solid research plan related to parallel GNN training.

2. He has two versions of parallel CNN implementations: bare TF and Horovod versions.
The Horovod version works better (as expected).

3. Due to the software restrictions, he padded the samples so that all the samples have the same size and then trained the model using TF.
I don't clearly understand why but it seems like that's the restriction coming from the bare TF parallel module.

4. We will be getting his TF implementation code soon. I asked the code and he said he will migrate the code to the current ExaTrkX repo.

5. They also have faced the memory footprint issue because of the large sample sizes. They even consider dividing the graphs to reduce the memory footprint.

Prof. Liao's thoughts:
Based on Sunwoo’s summary, I assume Xiangyang’s parallelization is
1. data parallelism
2. using TF and Horovod, nothing fancy (i.e. I/O and comm)
3. the same approach is considered by Jeremy?

Claire's thoughts:
1. Since Xiangyang has shared his TF implementation of training I believe my implementation using Pytorch could prove to be useful as a point of comparison/benchmark.

2. The reversible networks to reduce the memory footprint can be a solution to the problem. According to the paper, it performs similarly to message-passing networks but at a significantly-reduced memory footprint supporting larger scale graphs. In the future, we can try to implement this and see if it works well. The link to the paper is here: http://papers.nips.cc/paper/9510-graph-normalizing-flows.pdf



