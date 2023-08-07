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

## Installing Data

TODO limit of home dir, DBIC, and the ACL

## The Great and Evil ACL

TODO what is an ACL Link
TODO tldr ask yarik to (TODO give acl?)

If you drag&drop a file from your Windows OS machine into the mounted POSIX environment, its permissions now will be ACLed and you will not be able to do anything with that file until you use `nfs4_setfacl -e filename` to interactive remove "ace"s from the file permissions. (TODO: JH example)
TODO https://github.com/con/opfvta-replication-2023/issues/33
TODO https://rc.dartmouth.edu/wp-content/uploads/2019/04/Intro_to_Cluster.pdf

## POSIXy filesystem(s) for git-annex/DataLad inspired
TODO
