Zabbix template for monitoring Borg backup repositories. Requires Borg binary to be available in the system being monitored. Monitoring occurs on the backup server, though it can be possible to invoke this by the client and connect to repository via Borg network capabilities. This setup is currently not supported by the plugin.

# Installation
1. Copy `zabbix_agentd.d/borg.conf` to the Zabbix agent's configuration directory (usually located at `/etc/zabbix`). If using encrypted repositories, read the note below and use the file `zabbix_agentd.d/borg-enc.conf`.
2. Import template configuration `templates/borg.xml` to Zabbix web frontend.

# Notes
This plugin uses discovery capabilities provided by Zabbix to locate backup repositories. Since all commands are executed by `zabbix` user, appropriate permissions must be granted to all backup repositories. If encrypted, encryption keys should be provided to `zabbix` user.

One possibility is to use `--umask=0027` parameter when invoking `borg` command to make created files group readable. Then add the user `zabbix` into the group of the SSH account used by backup client. In this case, the backup repository must be created with `--umask=0007` to make it group writable so that `zabbix` user can create lock files while polling the backup status.

# Encrypted Repositories
For encrypted repositories, a `borg info` command must be run right after the backup and output saved to a file `status.txt` located in the remote repository. Also the output of the `borg check -v` should be appended to `status.txt`. Since those commands are run right after the backup by the client, they will contain the latest status of the repository. This plugin can then reside in the remote host and read the status from `status.txt`. This effectively avoids making encryption keys available to `zabbix` user neither on backup client or remote storage server.

To save the output of aforementioned commands, use something like:
```
	borg info user@host:repo::backup | ssh user@host 'cat > repo/status.txt'
	borg check user@host:repo | ssh user@host 'cat >> repo/status.txt'
```