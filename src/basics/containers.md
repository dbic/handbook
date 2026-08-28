# Containers

## ReproNim/containers

We highly recommend (re)using the collection of Singularity containers prepared by the ReproNim project, and ideally using the `datalad-container` extension.
See [a typical YODA workflow](https://github.com/ReproNim/containers#a-typical-yoda-workflow) for more information on how it should typically be used.

Besides using that collection via DataLad, you can use the images directly for the versions we have already prefetched (if any is missing, please let us know).
That might be preferable over fetching your own copies of the containers, so that no more space is wasted on Discovery (some containers are large).
For example, to validate your BIDS dataset using `bids-validator`, do:

```shell
cd path/to/my/bids/dataset
singularity run -B $PWD:$PWD /dartfs/rc/lab/D/DBIC/DBIC/archive/containers/images/bids/bids-validator--1.11.0.sing $PWD
```

## NeuroDocker

TODO

## ReproZip

TODO

## ReproMan

TODO
