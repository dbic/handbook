# Everything you need to know about Discovery HPC

## The Great and Evil ACL

If you drag&drop a file from your Windows OS machine into the mounted POSIX 
environment, its permissions now will be ACLed and you will not be able to do 
anything with that file until you use `nfs4_setfacl -e filename` to interactive 
remove "ace"s from the file permissions. (TODO: JH example)


## POSIXy filesystem(s) for git-annex/DataLad inspired

TODO

## Password less authentication for SSH

**Note**: ATM it first requires a regular login first, or otherwise even with
`kinit` just locally, it logs in but permissions are not properly set (TODO JH)

1. modify ~/.ssh/config to have a section like

        Host discovery7.hpcc.dartmouth.edu discovery7
          GSSAPIAuthentication yes

2. install kerberos client utilities.

On Debian: `sudo apt-get install krb5-user`

3. initialize your kerberos token

        kinit <NETID>@KIEWIT.DARTMOUTH.EDU
        
where `<NETID>` is your Dartmouth NetID (like `d11191d`).  Use `klist` command
to see if there is an active token.  As you will see from `klist` output, that 
token has expiration date, which is 10 hours from the moment you `kinit`ed it. 
You could use `kinit -R` (or just ssh again) to refresh the ticket.  It will be
refreshed for up to 30 days. 


TODO: make it even more sophisticated (auto-updated)

Now you should be able to just `ssh discovery7` (or `ndoli` -- TODO JH)
