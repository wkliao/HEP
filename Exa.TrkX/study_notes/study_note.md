## Study Note of HEP Particle Tracking Projects

* LHC experiment data consists of many slices, each corresponding to a detector layer.
  Slices are collected throughout a sequence of time. Each slice has a unique time stamp.
* Base on reference 2, the workflow of particle tracking consists of the following 4 stages.
  1. hit clustering
  2. seed finding
  3. track building
  4. track fitting

<p align="center">
<img align="center" src="3d.jpg" alt="3D plane detector toy data" width="600">
</p>

* **Hit clustering**
  + **Hit** -- A hit is a particle location detected on a slice (detector layer).
  + Data clustering algorithm is applied on each slice to identify individual
    particles from those pixels, so each particle is represented by its 3D coordinates.
  + Application of clustering on the raw data slice is necessary, because a particle
    may be captured by multiple detector wires and appear in multiple pixels, e.g. a
    pixel and its surrounding pixels.
  + There are O(10^8) pixels and *hit clustering* reduces it to O(10^5).
  + Hits are the vertices in the graph representation.
  + **ML methods used**
    * Shallow neural networks are used to identify shared cluster.
* **Seed finding**
  + **Seed** -- A seed is a tuple of 3 coordinates from 3 consecutive slices,
    also referred as *triplet*.
    * A seed can also be a *doublet*, i.e. from 2 hits on two consecutive slices.
    * A seed is the smallest set of space-points that allows to estimate the track curvature.
  + Seeds are selected based on the doublet/triplet geometry, its momentum, and its perigee.
  + **ML methods used**
    * **TODO**: to collect from papers
* **Track building**
  + Seeds found and constructed from the seed finding stage are just candidates of tracks.
  + Filtering out the "fake" seeds is required before building tracks.
  + **TODO**: need to collect more info about the remaining building process after filtering.
  + **ML methods used**
    * Combinatorial Kalman Filter (CKF) is a well-known method that is very efficient to filter out fake seeds
    * CKF is known to be computational expensive and not scalable.
    * RNN, LSTM, see reference 4. Given a sequence of hits, the model is built to predict the next hit.
      So, a track can be constructed/expanded by the sequence.

* **Track fitting**
  + A full resolution fit is performed on the final set of track candidates.
  + Each seed can have multiple candidate edges.
  + True edge must be identified, so the entire particle track can be constructed.
  + **ML methods used** that generate models to classify an edge into *true* and *false*.
    * CNN that takes images, i.e. pixels, for training and testing
    * LSTM that takes sequences of data points (coordinates)
    * GNN that takes graph data, i.e. V and E
      + Vertex set V is a collection of track candidates.
      + Edges? **TODO** How is the graph  constructed?
      + Message passing learning passes 'learning' around neighbors of a vertex
      + See reference 4.

## FermiLab Collaboration
* Problem Definition: Reconstruct trajectories of charged particles from collision event by correctly classify the path of the particle

* Input Dataset: DUNE HDF5 Dataset
  + Cori File Paths to Dataset: /global/cscratch1/sd/csl/hit2d
    * fluxswap dataset: /global/cscratch1/sd/csl/hit2d/fluxswap
    * nonswap dataset: /global/cscratch1/sd/csl/hit2d/nonswap

* Processed Dataset: Pytorch Dataset
  + Cori File Paths to Dataset: /global/cscratch1/sd/csl/processed

* Model output: Classification of true or false edges in particle reconstruction graph

## TrkX Project Home Pages
1. [Exa.TrkX](https://exatrkx.github.io) Project Home Page, including URL to github repo
2. [HEP.TrkX](https://heptrkx.github.io) Project Home Page, including URL to github repo

## Open data sets
1. [TrackML Particle Tracking Challenge](https://www.kaggle.com/c/trackml-particle-identification) --
   data from a collision between proton bunches at the Large Hadron Collider at CERN.
   The goal is to group the recorded measurements or hits for each event into tracks,
   sets of hits that belong to the same initial particle.

## References
1. [Exa.TrkX all-hands meeting](https://indico.fnal.gov/event/23974/), April 7, 2020.
2. [The HEP.TrkX Project: deep neural networks for HL-LHC online and offline tracking](https://lss.fnal.gov/archive/2017/conf/fermilab-conf-17-326-cd.pdf), 2017.
3. [Graph Neural Networks for Particle Reconstruction in High Energy Physics detectors](https://arxiv.org/pdf/2003.11603.pdf), 2020.
4. [Novel deep learning methods for track reconstruction](https://arxiv.org/abs/1810.06111), 2018
5. [Track Seeding and Labelling with Embedded-space Graph Neural Networks](https://arxiv.org/abs/2007.00149), 2020
6. [Graph Neural Networks for Reconstruction in DUNE](https://indico.fnal.gov/event/46736/contributions/203490/attachments/138073/172668/2020-12-04_CLARIPHY_talk.pdf), 2020

## Additional References from Alexandra, 2/11/21:
Project-Specific:
This proposal is from 2019 and provides a good overview of the goal of the program, and a short synopsis of the relevant physics: https://indico.physics.lbl.gov/event/898/contributions/3701/attachments/1929/2418/ExaTrkX_Proposal-243931.pdf 

Jeremy Hewes' presentation July 2020: https://indico.slac.stanford.edu/event/371/contributions/1213/attachments/533/856/2020-07-14_GNNs_for_reconstruction_in_LArTPCs.pdf

Particle Physics, Tracking, Detectors, Neutrinos, Fermilab:
Good overview of ATLAS tracking, with best description of track seeding I have seen so far:  https://indico.cern.ch/event/742090/contributions/3071860/attachments/1693222/2724898/Tracking_in_ATLAS.pdf

Physics-based overview of LArTPC projects at Fermilab: https://indico.cern.ch/event/724045/contributions/2990064/attachments/1653764/2646372/Asaadi_2018_TAMU.pdf

Popular science descritption of neutrinos, fun graphics, Super-Kamiokande experiment: (can access Scientific American through Northwestern Library SSO): https://www.scientificamerican.com/article/detecting-massive-neutrinos
