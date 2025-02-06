
## Password less authentication for SSH

1. modify ~/.ssh/config to have a section like:

        Host discovery7.hpcc.dartmouth.edu discovery7
	      User <netid>
          GSSAPIAuthentication yes
          GSSAPIDelegateCredentials yes

2. install kerberos client utilities.

     - On Debian: `sudo apt-get install krb5-user`
     - On MacOS with [Homebrew](https://brew.sh/): `brew install krb5`

3. initialize your kerberos token

        kinit <NETID>@KIEWIT.DARTMOUTH.EDU

where `<NETID>` is your Dartmouth NetID (like `d11191d`).  Use `klist` command to see if there is an active token.
As you will see from `klist` output, that token has expiration date, which is 10 hours from the moment you `kinit`ed it. 
You could use `kinit -R` (or just ssh again) to refresh the ticket.
It will be refreshed for up to 30 days. 

TODO: make it even more sophisticated (auto-updated)

Now you should be able to just `ssh discovery7` (or `ndoli` -- TODO JH)

