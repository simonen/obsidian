---
tags:
  - centos
---
`systemd` stuffs all logging in `journalctl`. Handles rotation automatically

Logging facility that includes all kernel and services messages from early boot to final shutdown. Managed by the `journald` daemon.

System messages are stored in the `/run` directory

* `systemd-journal`: 
	* Tightly integrated with `systemd`. 
	* Collects messages from the kernel, boot procedure and services, and writes them to an event **journal**. Stored in binary format. 
	* Queried by `journalctl` command. 
	* Can read all messages generated by `systemd` units. 
	* The journal is **NOT** persistent between reboots by default. 
	* The journal is temporarily stored in `/run/log/journal`
* `rsyslogd`: Enhanced version of `syslogd`. A service that takes care of managing centralized log files. Writes messages to different files in the `/var/log` directory that are persistent between boots. Offers centralized logging and filtering messages with modules. Such modules allows to store messages in a database. Allows to configure remote logging and log servers
* `auditd`: Provides an in-depth trace of what services, processes and users are doing. `SELinux` uses `auditd`.
* `direct write`: Some services write directly in logs, **not recommended**

- `journalctl` - gives detailed logging info
- `systemctl status UNIT` - overview of the most recent significant events logged by `systemd`

Standard Log Files:

- `/var/log/messages`: Generic log where most messages go
- `/var/log/dmesg`: Kernel messages
- `/var/log/secure`: Authentication-related messages
- `/var/log/boot.log`: System startup messages
- `/var/log/audit/audit.log`: Audit messages. `SELinux` writes here
- `/var/log/maillog`: Mail server messages
- `/var/log/httpd`: Logs by Apache Server, written directly, not through `rsyslog`

#### Systemd-journald

`man journalctl`

Configuration file
`/etc/systemd/journald.conf`

Service unit
`systemd-journald.service`

`Journalctl` displays messages from the machine boot on. Starts at the top by default, using **less** pager.  **SHIFT + G** to go the bottom. Q to exit

``` bash
journalctl
```

With no arguments, returns all messages since boot

Some `journalctl` filters:
- `-f`: Acts like the `tail -f` command. `-n LINES` works also.
- `-k`: Kernel messages only
- ` _UID=1000`: Shows messages related to the user with ID 1000
- `-u UNIT`: Show logs about a specific unit
- `-p "PRIORITY"`: Filter out messages of the priority `[emerg, alert, crit, err, warning, notice, info, debug]`
- `--list-boot`: Lists boot IDs
- `-b`: Messages from last boot
- `-b BOOT_ID`: Shows messages from a specific boot.  A negative boot number means number of boots prior to the most recent.

``` bash
journalctl -o verbose
```

```
	UNIT=sshd.service # systemd unit
    SYSLOG_IDENTIFIER=sshd
    _COMM=sshd # the name of the command that generated the log
    _EXE=/usr/sbin/sshd
    _SELINUX_CONTEXT=system_u:system_r:sshd_t:s0-s0:c0.c1023
    _SYSTEMD_CGROUP=/system.slice/sshd.service
    _SYSTEMD_UNIT=sshd.service
    _CMDLINE="sshd: kimchen [priv]"
```

`-F "_FIELD"`: List all possible data values the specified field present in the journal

``` bash
journalctl -F _COMM
```

```
dnf
anacron
logger
packagekitd
su
sshd
(systemd)
```

Search for log entries generated by the `sshd` command

``` bash
journalctl _COMM=sshd
```

```
[time] headoffice.ohio.cc sshd[1172]: Server listening on 0.0.0.0 port 22.
[time] headoffice.ohio.cc sshd[1172]: Server listening on :: port 22.
```

Fields can be combined with logical operators (space for AND) and (+ for OR)

To show messages of a specific period

``` bash
journalctl [--since, -S "START"] [--until, U "END(excluding)"]
```

Some possible time values

- `--since`: `yesterday, today, YYYY-MM-DD hh:mm:ss`. 
- `--until`: `"1 hour ago"`, `"5 min ago"`

Show all `sshd` log files until 13:04:25 with unit files

``` bash
journalctl -u sshd -U '13:04:25' -o with-files
```

```
[root@dnsserver ~]# journalctl -u sshd -U '13:04:25' -o with-unit
Sun ... EET dnsserver init.scope[1]: Starting OpenSSH server daemon...
Sun ... EET dnsserver sshd.service[891]: Server listening on 0.0.0.0 port 22.
Sun ... EET dnsserver sshd.service[891]: Server listening on :: port 22.
Sun ... EET dnsserver init.scope[1]: Started OpenSSH server daemon.
Sun ... EET dnsserver sshd.service[2158]: Accepted publickey
```

#### journald.conf

`man 5 journald.conf`

The `systemd` binary journal file is stored by default in:
`/run/log/journal`

To make `systemd-journald` logging persistent:

`/etc/systemd/journald.conf`
```
[Journal]
# storage types
Storage=auto: # The journal will be written on disk if dir /var/log/journal exists
Storage=volatile: stores logs in memory in the /run/log/journal dir. Unreadable
Storage=persistent: will create and store logs in /var/log/journal if it doesn't exist. Uses /run/log/journal when disk is not available during the initial stages of booting
Storage=none: disables all local logging. Gives the option to forward logging to remote logging server
```

Create the journal folder

``` bash
mkdir  /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/journal
```

Restart the service for changes to take effect

``` bash
systemctl restart systemd-journal-flush.service
```

A journal directory is created in `/var/log/journal`

```
[root@headoffice log]# tree /var/log/journal
/var/log/journal
└── 43103637d4bf4805aaea05637b4010a6
    └── system.journal
```

`/etc/systemd/journald.conf`
```
SystemMaxUse= : controls the size of the log storage on disk
RuntimeMaxUse= : controls the size of the volatile storage. Default is 10% to max 4 GB
SystemKeepFree= : controls how much disk space is kept free for other uses. Default is 15% and 4GB. 
RuntimeKeepFree= : N B, K, M, G, T
MaxRetentionSec= : controls how long log files are retained - year, month, week, day. '6 month'
```

##### Securing Journald with FSS

Forward Secure Sealing (FSS) signs the logs with of a generated key pair. A `sealing key` seals the logs at a specified interval, a `verify key` can be used to detect tampering.

``` bash
journalctl --setup-keys
```

```
New keys have been generated for host headoffice.ohio.cc/43103637d4bf4805aaea05637b4010a6.
...
c902de-90d01c-9279b3-89270c/1d5162-35a4e900 <- seal key
```

The signing key has been placed in the `fss` file in the journal directory

```
[root@headoffice log]# tree /var/log/journal
/var/log/journal
└── 43103637d4bf4805aaea05637b4010a6
    ├── fss
    └── system.journal
```

To check the journal file for internal consistency

``` bash
journalctl --verify
```

To specify a verification key

``` bash
journalctl --verify-key="SEAL KEY"
```

```
PASS: /var/log/journal/43103637d4bf4805aaea05637b4010a6/system.journal
```
#### Logger

The **logger** command allows users to write message into the system log.
Logger can specify the priority and facility messages are sent to.

``` bash
logger [options] "MESSAGE"
```

To specify priority. The default is `user.notice`. Can be specified as `facility.level`, level or number.

``` bash
logger -p "FACILITY"."LEVEL" "MESSAGE"
```
