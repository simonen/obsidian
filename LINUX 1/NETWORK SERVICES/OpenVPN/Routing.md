
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

The client can now ping the `10.0.6.2` address on the OpenVPN server but cannot access the `10.0.6.0/24` network yet.

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

#### Routing Subnets

VPN Server -> GW-A <-NAT NETWORK-> GW-B <- VPN CLIENT 

In this client/server setup, the client will be aware of internal networks behind the VPN server and be able to reach LAN Client 10.0.7.3

VPN Server: IP 10.0.7.2/24, Tunnel end: 10.200.0.1
LAN: 10.0.7.0/24 

LAN Host:  IP 10.0.7.3/24

GW-A: Public IP: 10.0.2.7, LAN: 10.0.7.1
GW-B: Public IP: 10.0.2.8, LAN: 10.0.6.2

VPN Client1: IP 10.0.6.3. Tunnel end: 10.200.0.2

##### Gateways

Enable forwarding and Static SNAT (Source NAT). This will rewrite the source address of packets coming from `10.0.7.0/24` to `10.0.2.7`

Enable forwarding on both routers

```gw-a,b
sysctl -w net.ipv4.ip_forward=1
```

Add NAT to both routers

`gw-a`
```bash
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -s \  10.0.7.0/24 -d 10.0.2.0/24 -o enp0s8 -j SNAT --to-source 10.0.2.7
```

`gw-b
```bash
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -s \  10.0.7.0/24 -d 10.0.2.0/24 -o enp0s8 -j SNAT --to-source 10.0.2.8
```

`-j MASQUERADE`: if dynamic IP.

Enable port forwarding to redirect incoming connection on the router:1194 to the VPN server:1194

`gw-a`
```bash
firewall-cmd --permanent --direct --add-rule ipv4 nat PREROUTING 0 -i enp0s8 -p \ udp --dport 1194 -j DNAT --to-destination 10.0.7.2:1194
```

The VPN client can now connect to the VPN Server at GW-A:1194 UDP.

To route VPN traffic (i.e., coming from `10.200.0.0/24`) back to the VPN server and then through the tunnel

`gw-a`
```bash
ip route 10.200.0.0/24 via 10.0.7.2 <- VPN server
```
##### VPN Server Settings

The OpenVPN host should be able to forward packets

```bash
sysctl -w net.ipv4.ip_forward=1
```

Push the route to the internal network to connecting clients

`server.conf`
```
push "route 10.0.7.0 255.255.255.0"
```

VPN  clients should now be able to access hosts on the `10.0.7.0/24` network behind the OpenVPN server

Optionally, Use `client-config-dir` to push it only to specific clients.

Filename of the client configuration must match the CN on client's certificate. 

`/etc/openvpn/server/ccd/client1`
```
iroute 10.0.6.0 255.255.255.0
```

Thе `iroute` directive tells OpenVPN that `10.0.6.0/24` is behind `client1` client, and OpenVPN will internally handle routing via the correct VPN tunnel. In this case - `10.200.0.2`
