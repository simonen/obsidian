
#### Limiting User Access

Add the **AllowUsers** *USER1 USER2* etc option in **/etc/ssh/sshd_config**

**MaxAuthTries**: for logging purposes. Does not block connections.

#### Other Useful sshd options

Session options

**UseDNS**: makes sure the resolved hostname maps back to the host's IP address
Could be the reason for slow client ssh connection
**MaxSessions**: max number of sessions that can connect to a ssh server from one IP. Default 10
#### Connection Keepalive Options

Server-side configurations

**TCPKeepAlive**: ensures that inactive connections are released
**ClientAliveInterval**: the interval in seconds after which the server sends a packet to the client if it is inactive
**ClientAliveCountMax**: how many of these packets should be sent

**ClientAliveInter** 30 and **ClientAliveCountMax** 10 means inactive connections connections are kept for about 5 minutes

Client-side options to send keepalive traffic to the server Can be set per user in ~/.ssh/config:
**ServerAliveInterval**
**ServerAliveCountMax**


