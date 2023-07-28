## ReproNim/containers

We highly recommend (re)using collection of Singularity containers prepared by ReproNim
project, and ideally using datalad-container extension.  See
https://github.com/ReproNim/containers#a-typical-yoda-workflow
for more information on how it should typically be used.

TODO: prepare local (discovery) specific instructions since we have a local clone with some
images/versions already available.

Besides using that collection via datalad, you can use them directly for the
images/versions we already prefetched (if any is missing, please let us know).
That might be preferable over manual fetching of container copies to not waste
more space on discovery (some containers are large). E.g. to validate
your BIDS dataset using bids-validator, do

    cd path/to/my/bids/dataset
    singularity run -B $PWD:$PWD /dartfs/rc/lab/D/DBIC/DBIC/archive/containers/images/bids/bids-validator--1.11.0.sing $PWD

## NeuroDocker
## ReproZip
## ReproMan 
