# Neuroimaging Data Access

All data is collected and provided for access and conversion into BIDS on rolando.cns.dartmouth.edu.

You need to request access permissions from [Andrew Connolly](mailto:andrew.c.connolly@dartmouth.edu).

## DICOMs

All dicoms are organized into `YEAR/MONTH/DATE/ACCESSION` hierarchy under `/inbox/DICOM`.
You can `scp` or `rsync` them to your local storage.

## BIDS

At the moment, upon request from a lab member to [Yaroslav Halchenko](mailto:yoh@dartmouth.edu), 
data is converted from DICOMs into BIDS within the directories hierarchy under `/inbox/BIDS`, following
convention described in ReproIn section (TODO: make into a reference).

These directories are also DataLad datasets, so you have two options on how to transfer them:

### DataLad way

Use [datalad install](TODO) command to obtain dataset, and then [datalad get]() to 
obtain specific files.  If you are greedy, add `-r` to get full hierarchy of datasets, and/or `-g`
to immediately also fetch all data files
  
    datalad install -s rolando.cns.dartmouth.edu:/inbox/BIDS/dbic/dbic-animals
    cd dbic-animals
    datalad get -J4 sub-*  # to get only converted data, without tarballs etc

Later upgrades to fetch new data (subjects etc) could be done via 

    datalad update --merge -r 


#### Specifics/workarounds

##### Discovery filesystem

Unfortunately the filesystem used on discovery by default does not support smooth git-annex and thus DataLad operation.
If you use `datalad install` or `datalad clone` as instructed above, you would likely to endup in "adjusted" git-annex branch which would complicate your interactions with the data, etc.
We recommend to use new feature of git-annex allowing for custom protection of data on discovery.
For that

**Step 1: make sure you are using recent git-annex**

Make sure that you are using recent (at least as of January 2022) version of git-annex.
For that you could use the version we provide and just adjust your `~/.bashrc` with the following content:

```shell
ANNEX_BIN_PATH=/dartfs/rc/lab/D/DBIC/DBIC/archive/git-annex/usr/lib/git-annex.linux/
echo $PATH | grep -q "$ANNEX_BIN_PATH" || export PATH="$ANNEX_BIN_PATH:$PATH"
```

**Step 2: configure git-annex to use custom data protection**

Adjust you global `~/.gitconfig` with the following section

```
[annex]
thawcontent-command = /dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/thaw-content %path
freezecontent-command = /dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/freeze-content %path
```

Now, whenever you `datalad install` data from rolando you should end up in `master` branch.

##### Parallel get - multiple passwords

If you are `get`ing data to discovery, to non-POSIX compliant filesystem, then you must provide
option `-J1` to `datalad get` to prevent parallel downloads and multiple password prompts.

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
    
    
You could add `--exclude=sourcedata` and/or `--exclude=derivatives` to exclude
folders with original DICOMS and possible derivatives (fmriqc, etc).
    
