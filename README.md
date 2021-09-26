# rdb3

Utility functions for running
[`rdiff-backup`](https://rdiff-backup.net/), see INSTALL.md for more
info on the configuration. The main application scenario is to backup
several clients to a single server. The server may off-load the
repository image to cloud storage.


## Why rdiff-backup?

Because I like it. It's main feature is that it stores
backward-deltas. This means, cleaning out old backups is simple. The
main scenario for restore is emergency, where I want my lates version
back.

The second feature I really like is the built-in client-server
architecture and the resulting security features. The clients can be
restricted to `update-only`. Many backup scenarios assume the client
has legit access to the data. Malware is not legit, but may have an
interest in deleting or tampering your backup repository. If you ever
considered backing up to an USB-disk, scrap that.

The second restricted access method is `read-only`. Also neat.

Does this fully protect? Nope. If I had a working concept for fully
protecting a system from malware, I'd sell this for a fortune. But it
decreases the size of the attack vector. Clients are kind of open,
full of thoughtless users and such. Servers can have a really confined
admin access. Having a dedicated server for backups-only is a luxury
you may or may not want to afford. Mine is a RasPi4 with one 2TB USB
attached.

But this goes to another unreliable disk? Yes, this is why my story
does not end here. The repository on the server is reverse mounted
with [`gocryptfs`](https://github.com/rfjakob/gocryptfs). The
encrypted image is `rsync'ed to cloud storage. I have paper-copies of
the encryption keys and the login to the cloud storage...
