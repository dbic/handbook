# Passwordless authentication for SSH

1. Modify `~/.ssh/config` to have a section like:

   ```ssh-config
   Host discovery.dartmouth.edu ndoli.dartmouth.edu
     User <netid>
     GSSAPIAuthentication yes
     GSSAPIDelegateCredentials yes

   Host discovery
     HostName discovery.dartmouth.edu

   Host ndoli
     HostName ndoli.dartmouth.edu
   ```

1. Install the Kerberos client utilities:

   - on Fedora: `sudo dnf install krb5-workstation`
   - on Debian: `sudo apt-get install krb5-user`
   - on macOS with [Homebrew](https://brew.sh/): `brew install krb5`

1. Initialize your Kerberos token:

   ```shell
   kinit <NETID>@KIEWIT.DARTMOUTH.EDU
   ```

   where `<NETID>` is your Dartmouth NetID (like `d11191d`).

Use the `klist` command to see whether there is an active token.
As you will see from the `klist` output, that token has an expiration date, which is 10 hours from the moment you `kinit`ed it.
You can use `kinit -R` (or just ssh again) to refresh the ticket.
It will be refreshed for up to 30 days.

TODO: make it even more sophisticated (auto-updated).

Now you should be able to just `ssh discovery` or `ssh ndoli`.
