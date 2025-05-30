
Server1
IP: 192.168.137.16
LAN: 10.0.6.2/24

Server2
IP: 192.168.137.10
LAN: 10.0.7.2/24

`server1`
```
dev tun
proto udp
local 192.168.137.16
lport 1194
remote 192.168.137.10
rport 1194

ifconfig 10.200.0.1 10.200.0.2
route 10.0.7.0 255.255.255.0

user nobody
group nobody

persist-tun
persist-key
keepalive 10 60
ping-timer-rem

verb 3
daemon
log-append /tmp/openvpn.log
```

`server2`
```
dev tun
proto udp
local 192.168.137.10
lport 1194
remote 192.168.137.16
rport 1194

ifconfig 10.200.0.2 10.200.0.1
route 10.0.6.0 255.255.255.0

user nobody
group nobody

persist-tun
persist-key
keepalive 10 60
ping-timer-rem

verb 3
daemon
log-append /tmp/openvpn.log
```

Now both VPN ends have the routes to the respective remote LANs and can ping the appropriate local network addresses

`server1`
```
10.0.7.0/24 via 10.200.0.2 dev tun0
10.200.0.2 dev tun0 proto kernel scope link src 10.200.0.1
```

`server2`
```
10.0.6.0/24 via 10.200.0.1 dev tun0
10.200.0.1 dev tun0 proto kernel scope link src 10.200.0.2
```

> The hosts on the 10.0.X.0/24 networks still have to have routes to the tunnel added manually

> TLS must be configured for encryption in production environments!

