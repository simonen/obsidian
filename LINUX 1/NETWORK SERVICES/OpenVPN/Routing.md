
#### Allow VPN Clients to Access Internal Network Behind VPN Server

The VPN clients should be able to access the 10.0.6.0/24 network that is local to the OpenVPN server.

1. OpenVPN server:
	VPN endpoint: 10.200.0.1/24
	LAN: 10.0.6.2

2. OpenVPN client:
	VPN endpoint: 10.200.0.2/24

3. Internal host:
	LAN: 10.0.6.11

`server`
```bash
openvpn --ifconfig 10.200.0.1 10.200.0.2 --dev tun
```

`client`
```
openvpn --ifconfig 10.200.0.2 10.200.0.1 --dev tun
```
##### Static Routes

`client`
```
Network Destination        Netmask          Gateway       Interface  Metric
          10.0.6.0    255.255.255.0       10.200.0.1       10.200.0.2     26
```

The client can now ping the 10.0.6.2 address on the OpenVPN server but cannot access the 10.0.6.0/24 network yet.

Enable IPv4 forwarding on the OpenVPN server

`server`
```
sysctl -w net.ipv4.ip_forward=1
```

Check the default forward policy in `iptables`

`server`
```bash
iptables -L FORWARD
```

If it is set to ACCEPT, this is enough. There is no need to configure `iptables`.

The packets arrive but the internal host does not know where to return them yet. Configure the internal host(s) to return packets to the VPN tunnel via the OpenVPN LAN address serving as the gateway.

`internal host`
```
10.200.0.0/24 via 10.0.6.2 dev enp0s8
```

Now the client can return packets to the internal host

To add the appropriate routes when the tunnel comes up

```
openvpn --ifconfig LOCAL_VPN_ENDPOINT REMOTE_VPN_ENDPOINT --dev tun0 \
--route INTERNAL_NETWORK MASK
```

```bash
openvpn --ifconfig 10.200.0.1 10.200.0.2 --dev tun0 \
--route 10.0.6.0 255.255.255.0
```

`--route 10.0.6.0 255.255.255.0` is the same as `route add 10.0.6.0/24 via 10.200.0.2`
routes the internal network through the remote end of the VPN tunnel.

