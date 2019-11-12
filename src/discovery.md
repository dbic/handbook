# Everything you need to know about Discovery HPC

Discovery is a HPC cluster which DBIC users could utilize to run heavy computation.
[Discovery Overview](https://rc.dartmouth.edu/index.php/discovery-overview/)
and pages within provide official generic information about Discovery - what it
provides and how to use it.  This section provides additional, typically DBIC 
users specific information and hints.

## The "modules"

TODO: brief intro into [modules](http://modules.sourceforge.net/) - the system
used a collection of available environments.

For the purpose of using DataLad, please use `python/3.7-Anaconda-datalad` 
module which you can enable via `module load python/3.7-Anaconda-datalad`: 

```bash
[d31548v@discovery7 ~]$ which datalad
/usr/bin/which: no datalad in (/dartfs-hpc/admin/opt/el7/intel/...
[d31548v@discovery7 ~]$ module load python/3.7-Anaconda-datalad 
[d31548v@discovery7 ~]$ which datalad
/optnfs/common/miniconda3-datalad/bin/datalad
```
 

## The Great and Evil ACL

If you drag&drop a file from your Windows OS machine into the mounted POSIX 
environment, its permissions now will be ACLed and you will not be able to do 
anything with that file until you use `nfs4_setfacl -e filename` to interactive 
remove "ace"s from the file permissions. (TODO: JH example)


## POSIXy filesystem(s) for git-annex/DataLad inspired

TODO

## Password less authentication for SSH

1. modify ~/.ssh/config to have a section like

        Host discovery7.hpcc.dartmouth.edu discovery7
          GSSAPIAuthentication yes
          GSSAPIDelegateCredentials yes

2. install kerberos client utilities.

On Debian: `sudo apt-get install krb5-user`

On MacOS with [Homebrew](https://brew.sh/): `brew install krb5`

3. initialize your kerberos token

        kinit <NETID>@KIEWIT.DARTMOUTH.EDU
        
where `<NETID>` is your Dartmouth NetID (like `d11191d`).  Use `klist` command
to see if there is an active token.  As you will see from `klist` output, that 
token has expiration date, which is 10 hours from the moment you `kinit`ed it. 
You could use `kinit -R` (or just ssh again) to refresh the ticket.  It will be
refreshed for up to 30 days. 


TODO: make it even more sophisticated (auto-updated)

Now you should be able to just `ssh discovery7` (or `ndoli` -- TODO JH)
