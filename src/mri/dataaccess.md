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

Make sure you have git configured, in that it knows about you.
If you have output to running `git config user.name` command, then most likely you are all set.
If it comes out empty, please do


    git config --global user.name "First Last"
    git config --global user.email "someemail@address"

while replacing those values with your name and email.

If you are doing it to discovery HPC, please first checkout the next section of this documentation below (Discovery filesystem) on how to configure your git for Discovery's ACL filesystem, unless you know that it is pure POSIX.

Then it is recommended to create a directory for your study first, e.g. `mkdir ID_name` where `ID_name` are the same as on rolando, e.g. `0001_dbic-animals`, and `cd` into it, e.g.:

    mkdir 0001_dbic-animals
    cd 0001_dbic-animals 

Use [datalad install](https://docs.datalad.org/en/stable/generated/man/datalad-install.html) command
to obtain dataset, and then [datalad get](http://docs.datalad.org/en/stable/generated/man/datalad-get.html) to 
obtain specific files.  If you are greedy, add `-r` to get full hierarchy of datasets, and/or `-g`
to immediately also fetch all data files
  
    datalad install -s your-login-on-rolando@rolando.cns.dartmouth.edu:/inbox/BIDS/dbic/0001_dbic-animals dbic
    cd dbic
    datalad get -J4 sub-*  # to get only converted data, without tarballs etc

Later upgrades to fetch new data (subjects etc) could be done via 

    datalad update --how merge -r 


#### Specifics/workarounds

##### Discovery filesystem

Unfortunately the filesystem used on discovery by default does not support smooth git-annex and thus DataLad operation.
If you use `datalad install` or `datalad clone` as instructed above, you would likely to endup in "adjusted" git-annex branch which would complicate your interactions with the data, etc.
We recommend to use new feature of git-annex allowing for custom protection of data on discovery.
For that

**Step 1: make sure you are using recent git-annex**

Make sure that you are using recent (at least as of January 2023) version of git-annex.
For that you could use the version we provide and just adjust your `~/.bashrc` with the following content:

```shell
ANNEX_BIN_PATH=/dartfs/rc/lab/D/DBIC/DBIC/archive/git-annex/usr/lib/git-annex.linux/
echo $PATH | grep -q "$ANNEX_BIN_PATH" || export PATH="$ANNEX_BIN_PATH:$PATH"
```

So whenever you re-login (or open a new `bash`) and type `git annex version` you should get version past above date.

**Step 2: configure git-annex to use custom data protection**

Adjust you global `~/.gitconfig` with the following section

```
[annex]
thawcontent-command = /dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/thaw-content %path
freezecontent-command = /dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/freeze-content %path
```

which also could be done via running commands

```shell
git config --global annex.thawcontent-command '/dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/thaw-content %path'
git config --global annex.freezecontent-command '/dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/freeze-content %path'
```

**Step 3: make sure that directory has group ACL to remove children**

It is the [`D` ACE Permission](https://www.osc.edu/book/export/html/4523): if folder lacks it, then `git-annex` will be unable to move read-only file under `.git/annex`.
So, if you get a "Permission error" while trying to `git annex add` or `datalad save`, you might need to add that to the group permissions.
Use `/dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/fix-dir-group-perm` script with the folder under which you want to create/clone repo to add that `D`.

Now, after these 3 steps, whenever you `datalad install` data from rolando you should end up in `master` branch.
If that doesn't happen - file an issue.

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
    
