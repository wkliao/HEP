
# Average Gradients GNN Meeting Summary (August 20, 2020)
This file summarizes the key points made during the meeting with Jim, Jeremy, and Giuseppe during today's meeting

## Current Implementation
	* Multi-GPU Multi-process implementation of GNN code 
	* One process per GPU machine
	* Average gradients function used to share gradients among machines

## Issues and Future Research direction 
	* I/O is bottleneck, taking up 80-85% of the total epoch time
	* Use HDF5 concatenated input files to decrease I/O time
	* Run on different GPU nodes to compare I/O time
	* Investigate Cori Lustre file system for increased bandwidth (i.e. striping)

## Meeting Attendees
Jim, Giuseppe, Jeremy, Claire


## Afterthoughts and Action Items
1. Use h5py library to replace Pytorch input files with HDF5 concatenated input files
2. Change Cori batch script to use more than 1 GPU node
3. Look into Cori Lustre file system striping settings and maximum bandwidth 



