# Neuroimaging Data Access

All data is collected and provided for access and conversion into BIDS on rolando.cns.dartmouth.edu.

You need to request access permissions from [Andrew Connolly](mailto:andrew.c.connolly@dartmouth.edu).

## DICOMs

All dicoms are organized into `YEAR/MONTH/DATE/ACCESSION` hierarchy under `/inbox/DICOM`.
You can `scp` or `rsync` them to your local storage.

## BIDS

At the moment, upon request from a lab member to [Yaroslav Halchenko](mailto:yoh@dartmouth.edu), data is converted from DICOMs into BIDS within the directories hierarchy under `/inbox/BIDS`, following convention described in ReproIn section (TODO: make into a reference).

These directories are also DataLad datasets, so you have two options on how to transfer them:

##### Reckless clone still wants to access rolando

TODO: Yarik figureout

```
(conda-20200210-datalad) [d31548v@discovery7 tmp]$ datalad install -s 1021_actions --reckless auto 1021_actions_reckless
[INFO   ] Fetching updates for <Dataset path=/dartfs/rc/lab/D/DBIC/DBIC/tmp/1021_actions_reckless>


    Dartmouth College, Department of Psychological and Brain Sciences
                      Authorized access only




yohtest@rolando.cns.dartmouth.edu's password:
```

### Old fashion way

`scp` or `rsync`. But you would need to take care about dereferencing symlinks.

    rsync --exclude=.git --copy-links -r \
        rolando.cns.dartmouth.edu:/inbox/BIDS/dbic/dbic-animals dbic-animals


You could add `--exclude=sourcedata` and/or `--exclude=derivatives` to exclude folders with original DICOMS and possible derivatives (fmriqc, etc).

