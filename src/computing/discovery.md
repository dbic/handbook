# Everything you need to know about Discovery HPC

Discovery is an HPC cluster which DBIC users can utilize to run heavy computation.
The [Discovery Overview](https://rc.dartmouth.edu/index.php/discovery-overview/) and the pages within it provide the official generic information about Discovery -- what it provides and how to use it.
This section provides additional information and hints that are typically specific to DBIC users.

## Getting access

- See the [Research Computing page on cluster access](https://rc.dartmouth.edu/discovery-overview/accessing-the-cluster/)

## Must knows

- Please be considerate about the nodes you are using.
  When you log in you are on a login node, and no work should be done there!
  Instead, use the interactive node `x01`, the scheduling node `s01`, or, if you have permission, the fancy IT node `ndoli`.

- Home directories are limited to 50 GB of storage; for large datasets, use `/dartfs/rc/lab/D/DBIC/DBIC/`

## Recommended .bashrc

```bash
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# User specific aliases and functions

# Install conda
source /optnfs/common/miniconda3/etc/profile.d/conda.sh

# use DBIC-installed git-annex
# https://dbic-handbook.readthedocs.io/en/latest/computing/discovery.html#about-filedirectory-permissions-and-acls
ANNEX_BIN_PATH=/dartfs/rc/lab/D/DBIC/DBIC/archive/git-annex/usr/lib/git-annex.linux/
echo $PATH | grep -q "$ANNEX_BIN_PATH" || export PATH="$ANNEX_BIN_PATH:$PATH"

export TERM=xterm
export EDITOR=vim

alias dog="pygmentize -g"
```

## Installing software

Make sure that `datalad --version` reports at least 0.19.3.

### Containers

Notes on why and how to use containers on Discovery can be found in the
[Research ITC containers repo](https://git.dartmouth.edu/research-itc-public/containers-for-hpc/).

### Conda

TODO

### Modules

TODO: brief intro to [modules](http://modules.sourceforge.net/) -- the system used to manage the collection of available environments.

For the purpose of using DataLad, please use the `python/3.7-Anaconda-datalad` module, which you can enable via `module load python/3.7-Anaconda-datalad`:

```bash
[d31548v@discovery7 ~]$ which datalad
/usr/bin/which: no datalad in (/dartfs-hpc/admin/opt/el7/intel/...
[d31548v@discovery7 ~]$ module load python/3.7-Anaconda-datalad
[d31548v@discovery7 ~]$ which datalad
/optnfs/common/miniconda3-datalad/bin/datalad
```

## POSIXy filesystem(s) for git-annex/DataLad

TODO

## Installing data

TODO: limits of the home directory, DBIC storage, and the ACL.

Unfortunately the filesystem used on Discovery by default does not support smooth git-annex, and therefore DataLad, operation.
If you use `datalad install` or `datalad clone` as instructed above, you would likely end up on an "adjusted" git-annex branch, which would complicate your interactions with the data.
We recommend using the git-annex feature that allows for custom protection of data on Discovery.

For that:

### Step 1: make sure you are using a recent git-annex

Make sure that you are using a recent (at least as of January 2023) version of git-annex.
For that you could use the version we provide, by adjusting your `~/.bashrc` with the following content:

```shell
ANNEX_BIN_PATH=/dartfs/rc/lab/D/DBIC/DBIC/archive/git-annex/usr/lib/git-annex.linux/
echo $PATH | grep -q "$ANNEX_BIN_PATH" || export PATH="$ANNEX_BIN_PATH:$PATH"
```

(TODO: note that this setting needs to come after the venv activation.)

So whenever you re-login (or open a new `bash`) and type `git annex version`, you should get a version dated after the date above.

### Step 2: configure git-annex to use custom data protection

Adjust your global `~/.gitconfig` with the following section:

```ini
[annex]
thawcontent-command = /dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/thaw-content %path
freezecontent-command = /dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/freeze-content %path
```

which could also be done by running the commands

```shell
git config --global annex.thawcontent-command '/dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/thaw-content %path'
git config --global annex.freezecontent-command '/dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/freeze-content %path'
```

### Step 3: make sure that the directory has a group ACL to remove children

(See also the section below on ACLs for more background.)

It is the [`D` ACE permission](https://www.osc.edu/book/export/html/4523): if a folder lacks it, then `git-annex` will be unable to move a read-only file under `.git/annex`.
So, if you get a "Permission error" while trying to `git annex add` or `datalad save`, you might need to add that to the group permissions.
Use the `/dartfs/rc/lab/D/DBIC/DBIC/archive/bin-annex/fix-dir-group-perm` script on the folder under which you want to create/clone the repository to add that `D`.

Now, after these 3 steps, whenever you `datalad install` data from rolando you should end up on the `master` branch.
If that does not happen, file an issue.

### Parallel get -- multiple passwords

If you are `get`ing data to Discovery, to a non-POSIX-compliant filesystem, then you must provide the option `-J1` to `datalad get` to prevent parallel downloads and multiple password prompts.

## About file/directory permissions and ACLs

The traditional/legacy permission structure on Linux is a "user-group-other" triple, with three permission settings for each: "read-write-execute" (coded as rwx).
If you run `ls -l` on a file or directory, this is the core of what you see on the left, e.g. `rwxrwx---` would indicate that both the user and the group (both also specified in the `ls -l` "long" output) have full "read-write-execute" permissions, but others have none.

However, filesystems (including the DartFS filesystem on Discovery) can use "access control lists" (ACLs) to provide an alternative means of permission settings --- and ACLs can render the basic permission listing incomplete, if not incorrect (or at least capable of misleading).
Here are the key points:

- When an ACL is present there is a `+` on the `ls -l` permissions block
- ACLs allow for more than one group to have permissions associated with a file or directory
- On Discovery the `ls -l` output will show `rwx` in the legacy group permission bits if **any** group has `rwx`, not specifically the "primary" group listed (making the group + group permissions combination shown potentially "wrong")

To view ACLs the standard command is `getfacl`, but on NFSv4 filesystems (such as DartFS) the right version of that is `nfs4_getfacl`... and really the best option on Discovery is the locally provided wrapper `listacl`.

### ACL pro tips

- The local command `listADgroup` can provide a listing of group members in any ACL group by executing an Active Directory query (this is a Python wrapper that does an LDAP lookup and formats it, along with extra information about each member)
- Refer to the Research Computing docs for complete details --- [this doc](https://services.dartmouth.edu/TDClient/1806/Portal/KB/ArticleDet?ID=88459) on DartFS lab permissions is a good starting point (searching inside of [services.dartmouth.edu](https://services.dartmouth.edu) for "DartFS permissions" will show a few other locally generated documents)
