
#### rsyslogd Modules

modules add to **rsyslog** functionality


**syslog**: the original logging solution for UNIX. 
**rsyslog**: enhanced version of syslog. Backward compatible

##### Connecting journald to rsyslog

in **/etc/systemd/journald.conf** add:
```
$OmitLocalLogging on
$IMJournalStateFile imjournal.state
```
