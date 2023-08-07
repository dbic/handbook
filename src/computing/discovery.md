# Everything you need to know about Discovery HPC

Discovery is a HPC cluster which DBIC users can utilize to run heavy computation.
[Discovery Overview](https://rc.dartmouth.edu/index.php/discovery-overview/) and pages within provide official generic information about Discovery - what it provides and how to use it.
This section provides additional, typically DBIC users specific information and hints.

## Getting Access

## Installing Software

### Containers
### Conda
### "modules"

TODO: brief intro into [modules](http://modules.sourceforge.net/) - the system used a collection of available environments.

For the purpose of using DataLad, please use `python/3.7-Anaconda-datalad` module which you can enable via `module load python/3.7-Anaconda-datalad`: 

```bash
[d31548v@discovery7 ~]$ which datalad
/usr/bin/which: no datalad in (/dartfs-hpc/admin/opt/el7/intel/...
[d31548v@discovery7 ~]$ module load python/3.7-Anaconda-datalad 
[d31548v@discovery7 ~]$ which datalad
/optnfs/common/miniconda3-datalad/bin/datalad
```


## POSIXy filesystem(s) for git-annex/DataLad inspired
TODO

## Installing Data
TODO limit of home dir, DBIC, and the ACL

Unfortunately the filesystem used on discovery by default does not support smooth git-annex and thus DataLad operation.
If you use `datalad install` or `datalad clone` as instructed above, you would likely to endup in "adjusted" git-annex branch which would complicate your interactions with the data, etc.
We recommend to use new feature of git-annex allowing for custom protection of data on discovery.

For that:

**Step 1: make sure you are using recent git-annex**

Make sure that you are using recent (at least as of January 2023) version of git-annex.
For that you could use the version we provide and just adjust your `~/.bashrc` with the following content:

```shell
ANNEX_BIN_PATH=/dartfs/rc/lab/D/DBIC/DBIC/archive/git-annex/usr/lib/git-annex.linux/
echo $PATH | grep -q "$ANNEX_BIN_PATH" || export PATH="$ANNEX_BIN_PATH:$PATH"
```

(TODO note that setting this needs to be after the venv activation)

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

If you are `get`ing data to discovery, to non-POSIX compliant filesystem, then you must provide option `-J1` to `datalad get` to prevent parallel downloads and multiple password prompts.
