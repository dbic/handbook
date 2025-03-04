# Everything you need to know about Discovery HPC

Discovery is a HPC cluster which DBIC users can utilize to run heavy computation.
[Discovery Overview](https://rc.dartmouth.edu/index.php/discovery-overview/) and pages within provide official generic information about Discovery - what it provides and how to use it.
This section provides additional, typically DBIC users specific information and hints.

## Getting Access

- See the [Research Computing page on cluster access](https://rc.dartmouth.edu/discovery-overview/accessing-the-cluster/)

## MUST KNOWs

- Please be considerate about the nodes you are using. When you login, you
  are on a login-node, but no work should be done here! Instead, use an interactive node `x01`, scheduling node, `s01`, or if you have permission, the fancy IT node `ndoli`.
- Home dirs are limited to 50 GB storage; for large datasets, use `/dartfs/rc/lab/D/DBIC/DBIC/`

## Recommended .bashrc

```
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# User specific aliases and functions

# Install conda
source /optnfs/common/miniconda3/etc/profile.d/conda.sh

# use DBIC-installed git-annex
# https://dbic-handbook.readthedocs.io/en/latest/mri/dataaccess.html#discovery-filesystem
ANNEX_BIN_PATH=/dartfs/rc/lab/D/DBIC/DBIC/archive/git-annex/usr/lib/git-annex.linux/
echo $PATH | grep -q "$ANNEX_BIN_PATH" || export PATH="$ANNEX_BIN_PATH:$PATH"

export TERM=xterm
export EDITOR=vim

alias dog="pygmentize -g"
```

## Installing Software

Make sure `datalad --version` > 0.19.3

### Containers

Notes for why and how to use containers on Discovery can be found in the
[Research ITC containers repo](https://git.dartmouth.edu/research-itc-public/containers-for-hpc/).

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

(see also the section below on "ACL"s for more background)

It is the [`D` ACE Permission](https://www.osc.edu/book/export/html/4523): if folder lacks it, then `git-annex` will be unable to move read-only file under `.git/annex`.
So, if you get a "Permission error" while trying to `git annex add` or `datalad save`, you might need to add that to the group permissions.
Use `/dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/fix-dir-group-perm` script with the folder under which you want to create/clone repo to add that `D`.

Now, after these 3 steps, whenever you `datalad install` data from rolando you should end up in `master` branch.
If that doesn't happen - file an issue.

##### Parallel get - multiple passwords

If you are `get`ing data to discovery, to non-POSIX compliant filesystem, then you must provide option `-J1` to `datalad get` to prevent parallel downloads and multiple password prompts.

## About File/Directory Permissions and ACLs

The traditional/legacy permission structure on Linux is a "user-group-other" triple, with three permission settings for each: "read-write-execute" (coded as rwx). If you run `ls -l` on a file or directory, this is the core of what you see on the left, e.g. `rwxrwx---` would indicate that both user and group (both also specified in the `ls -l` "long" output) have full "read-write-execute" permissions, but others have none.

However, filesytems (including the DartFS filesyste on Discovery) can use "access control lists" (ACLs) to provide an alternate means of permission settings --- and ACLs can render the basic permission listing incomplete, if not incorrect (or at least capable of misleading). Here are key points:

- When an ACL is present there is a `+` on the `ls -l` permissions block
- ACLs allow for more than one group to have permissions associated with a file or directory
- On Discovery the `ls -l` output will show `rwx` in the legacy group permission bits if **any** group has `rwx`, not specifically the "primary" group listed (making group + group permissions combo shown potentially "wrong")

To view ACLs the standard command is `getfacl`, but on NFS4 fileystems (such as DartFS) the right version of that is `nfs4_getfacl`... and really the best option on Discovery is the locally provided wrapper `listacl`. 

### ACL Pro tips:

- The local command `listADgroup` can provide a listing of group members in any ACL group by executing an Active Directory query (this is a Python wrapper that does an LDAP lookup and formats it, along with extra information about each member)
- Refer to Research Computing docs for complete details --- [this doc](https://services.dartmouth.edu/TDClient/1806/Portal/KB/ArticleDet?ID=88459) on DartFS lab permissions is a good starting point (searching inside of [service.dartmouth.edu](service.dartmouth.edu) for "DartFS permissions" will show a few other locally-generated documents)
