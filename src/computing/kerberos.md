# Passwordless authentication for SSH

1.  Modify `~/.ssh/config` to have a section like:

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

1.  Install the Kerberos client utilities:

    - on Fedora: `sudo dnf install krb5-workstation`
    - on Debian: `sudo apt install krb5-user`
    - on macOS with [Homebrew](https://brew.sh/): `brew install krb5`

1.  Make sure your `ssh` client itself is built with GSS-API support.

    The `GSSAPI*` options above are only honoured by an `ssh` that was built
    against GSS-API.  A client without it quietly ignores them and falls back
    to asking for your password, which looks just like Kerberos "not working".
    `ssh -Q kex | grep gss` lists the GSS-API key exchange methods if your
    client has that support, and prints nothing if it does not.

    On Debian this now needs an extra package:

    ```shell
    sudo apt install openssh-client-gssapi
    ```

    Debian is splitting GSS-API out of `openssh-client`.  As of Debian 13
    (trixie) `openssh-client-gssapi` is still an empty package that only
    depends on `openssh-client`, so installing it changes nothing yet, but its
    package description states that "future releases will remove GSS-API
    support from openssh-client, so users who need it should install this
    package".  On Debian 12 (bookworm) it is available from
    `bookworm-backports`.

1.  Initialize your Kerberos token:

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
