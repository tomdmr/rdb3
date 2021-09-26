# Prerequisites

This package tries to simplify the deployment of `rdiff-backup`. The
main scenario is several independent clients dumping their backups
into one server machine. The assumption is, that all machines trust
each other, so the backup in the clear is OK. The transport, however,
is secured by encrypted `ssh` connection. For this scenario servernd
client have to be configured and run.

In case backups are stored only locally, you just need the client and
plenty of space. And an idea, how to get the backups to some safe
space.

For the client-server scenario, `ssh' is required and configured to be
used also password-less. The clients identify to the server with ssh-keys.

On the server, you need enough disk-space, a separate user account to
receive the data, and a cron-job to clean out old backups. In my case
it's a RasPi 4 with a 2TB USB-disk attached. How this is off-loaded
encrypted to the cloud is another story...

`rdiff-backup` version 2.0 requires Python 3.0, so that is another
prerequisite.

# General concept

The backup is triggered by either a cron- or an anacron-job. The executing user mus have rights to read all directories it wants to backup. If the backup goes to a local directory, the user needs also rw-access. This is why I prefer remote backups. 

For remote backups, the client starts an `rdiff-backup` instance on
the server. On the server-side, access rights are restricted. The
server always starts `rdiff-backup`, and some extzra parameters can be
applied. Adding `--restrict-update-only` rejects all delete
requests. This means, the malware that is encrypting all your file can
back them up, but cannot delete the still intact versions.

This also implies that the `--remove-older-than`-command can only be
issued by the server, never by the client. A cron-job on the server
must take care of this.

# Install client

There is not much installation required. You need Python 3,
'rdiff-backup' somewhere, and the client script, `rdb3'.

If you want a signalling mechanism for success/failure, you will need
some program to send alerts. I am using `go-sendxmpp`to text to my
prosody server. sSMTP or some SMS bridge may also work.


## Configure client
### Logging

### General Section


### Job section


# Install server

For the server install you need a mostly unpriviliged user. More or
less the only thing this user should have is to write below the
repository directory tree, execute rdiff-backup, and the server
program. The server program is a tiny python script that launches
rdiff-backup with the restricted directory for each
client. rdiff-backup can be launched on the server side with the
options `--restrict path`, `--restrict-read-only path`, or
`--restrict-update-only path`. The normal mode should be update only. 

## Client access

The client accesses the server by SSH. For this, we need a passwordless SSH-keypair. It is generated on the client like this:

```bash
  ssh-keygen -t ed25519 -N '' -f key_for_backup
```

This creates two file. `key_for_backup`remains on the client. By
convention, it should go into the directory `~/.ssh` of the user
performing the backu-up. When using ed-25519, the file
`key_for_backub.pub` contains the public key, looking similar to this one:

```bash
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGSV1hwhjiw3+A80dYv+w4bQl6gn8AQm2wneFCfcd37N user@machine
```

This must be copied to the server, into the home directory of the user running the backup process.




## Server configuration


### Create Backup user

### Configure ssh-access

```bash
command="/var/rdiffbackup/rdb3/rdbs machine1",no-port-forwarding,no-x11-forwarding,no-agent-forwarding ssh-ed25519 AAAAC...7N user@machine
```

This restricts access to only running the Python script `rdbs` and
disallows any port forwarding. This is about as save as it can get.

### Configure repository and config-file

This example shows some of the features of the Python ini-file
handling mechanism.  In the first section we can define values to be
reused in later sections. In this case, the path to the first
repository where all backups are stored.


```
[DEFAULT]
repo1    = /media/disk/rbackup
# Machine names
[machine]
repository      =  %(repo1)s/machine1/
#newsubsok = True
[anothermachine]
repository      =  %(repo1)s/another/
#newsubsok = True
[specialmachine]
repository      = /media/davfs/whatever/
```

This value is referred to in the example sections `[machine1]` and
`[anothermachine]`. In the sections, only to settings are
used. `repository` points to the specicific location of the backups
for this machine. All jobs go below this directory.  The second
setting is the logical value `newsubsok`, with a default of
`False`. If `True`, the client can create new subdirectories in the
repository. This is required when you add new jobs to a client
configuration. As a side effect this launches the server side
rdiff-backup job with the switch `--restrict repository`. This setting
is recommended only for the first run. Usually this should be set to
`False`. This launches `rdiff-backup` with the switch
`--restrict-update-only repository`, giving another line of corruption
defense. With this setting, the client can not delete or corrupt older
backups.
