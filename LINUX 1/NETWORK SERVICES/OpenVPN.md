---
tags:
  - openvpn
  - certificates
  - openssl
  - centos
---

Open Virtual Private Networks

Opens an SSL/TLS encrypted tunnel between two networks at separate physical locations
Dependent on openSSL

**Certificate Authority**

A **CA** issues digital certificates and certifies ownership of public keys
Using CA eliminates the need for keeping client certificates on the OpenVPN server

**TUN / TAP**

Virtual network interfaces built into the Linux Kernel. Specified in the client and server configuration files

**TUN**: for routed networks
**TAP**: for bridged networks

#### Installing Server and Client OpenVPN

Packages:
- `epel-release` (repo)
- `openvpn`: provides client and server functionality
- `NetworkManager-openvpn-gnome`: for GNOME systems

Sample configurations:
* `/usr/share/doc/openvpn/sample`

#### Setting up a Connection Test

**openVPN** works with **UDP** on port **1194** by default

Add the **openvpn** service to the firewall on the **openvpn** server

```
firewall-cmd --add-service=openvpn --permanent ; firewall-cmd --reload
```

Create a basic VPN tunnel from Host1 to Host2
`Host 1`
``` bash
openvpn --dev tun0 --ifconfig 10.0.0.1 10.0.0.2 [--daemon]
```

Create a VPN tunnel from Host2 to Host1
`Host 2`
``` bash
openvpn --remote 'Host1' --dev tun0 --ifconfig 10.0.0.2 10.0.0.1 [--daemon]
```

`--dev tun`: creates a layer 3 VPN tunnel
`--ifconfig <local VPN endpoint> <remote VPN endpoint>`
`--daemon`: sends the job to the background

This will set up a tunnel between the hosts. Their current sessions will be taken by the tunneling, so open new sessions to use the hosts.

`Host2`
```
net_addr_ptp_v4_add: 10.8.0.2 peer 10.8.0.1 dev tun0
TCP/UDP: Preserving recently used remote address: [AF_INET]10.0.2.155:1194
UDP link local (bound): [AF_INET][undef]:1194
UDP link remote: [AF_INET]10.0.2.155:1194
Peer Connection Initiated with [AF_INET]10.0.2.155:1194
Initialization Sequence Completed
```

`Host1`
```
TUN/TAP device tun0 opened
/sbin/ip link set dev tun0 up mtu 1500
/sbin/ip addr add dev tun0 local 10.8.0.1 peer 10.8.0.2
UDPv4 link local (bound): [AF_INET][undef]:1194
UDPv4 link remote: [AF_UNSPEC]
Peer Connection Initiated with [AF_INET]10.0.2.156:1194
Initialization Sequence Completed
```
#### Setting Up Encryption with Pre-shared Static Keys

Create the static key in `/etc/openvpn/keys`

``` bash
openvpn --genkey --secret 'FILE.key'
```

Copy it over to the other host.

`/etc/openvpn/server.conf`
```
# server.conf
dev tun
ifconfig 10.0.0.1 10.0.0.2
secret /etc/openvpn/keys/FILE.key
local LOCAL_IP_ADDR
cipher AES-256-CBC # for stronger encryption
```

`/etc/openvpn/client.conf`
```
# client.conf
dev tun
ifconfig 10.0.0.2 10.0.0.1
secret /etc/openvpn/keys/FILE.key
remote SERVER_IP_ADDR
cipher AES-256-CBC
```

Start the connection on both, server and client
- `openvpn /etc/openvpn/server.conf`
- `openvpn /etc/openvpn/client.conf`

This type of connection is limited to two hosts only

#### Installing EasyRSA to Manage PKI

[[LINUX/PKI/OpenSSL]] For certificate generation using `openssl`

Download the easy-rsa package from here and unzip it
https://github.com/OpenVPN/easy-rsa/releases/

To add the easyrsa directory to the PATH variable, add the following to the .bashrc file

```
export PATH="/EASYRSA_DIR:$PATH"
```

Create an empty structure for the PKI

``` bash
easyrsa init-pki
```

Build the Certificate Authority. the CA creates and signs server and client certificates

``` bash
easyrsa build-ca
```

Pick strong passphrases to protect it. Choose a **Common Name**
This will create the **ca.crt** and **ca.key** in **EASYRSA_DIR/pki**

Generate a **keypair** and a **certificate signing request** for the OpenVPN server. Ignore protecting the keys with a passphrase by using the **nopass** option

``` bash
easyrsa gen-req 'VPNSERVER_CN' nopass
```

Generate a keypair and a certificate signing request for the client. Put a pass on it

``` bash
easyrsa gen-req 'CLIENT_CN' nopass
```

Sign the requests using their **Common Names**. Server 

``` bash
easyrsa sign-req server 'VPNSERVER_CN'
```

Sign the client requests

``` bash
easyrsa sign-req client 'CLIENT_CN'
```

Generate the Diffie-Hellman parameters for the server -> dh.pem

```
easyrsa gen-dh
```

Create a Hash Based Authentication Code (HBAC) key on the OpenVPN server. `tls-auth`

``` bash
openvpn --genkey --secret ta.key
```

Certificate Distribution
* **vpn server**: ca.crt, VPNSERVER_CN.crt, VPNSERVER_CN.key, dh.pem, ta.key
* **vpn client**: ca.crt, CLIENT_CN.crt, CLIENT_CN.key, ta.key

The location of the certificates on the machines is not strict and is defined in the .conf files
The .req files are no longer necessary

**HBAC**: verifies the integrity of messages
#### Creating and Testing Server and Client Connections

Sample conf files:
- `/usr/share/doc/openvpn/sample/sample-config-files`

>No `openvpn` daemons should be running

`/etc/openvpn/server.conf`
```
# Connection

dev tun
port 1194
proto udp
topology subnet
server <TUNNEL_NETWORK> <SUBNET_MASK>
keepalive 10 120

# Logging

log-append /var/log/openvpn.log
status /var/log/openvpn-status.log
verb 4
mute 20

# Security

user nobody
group nobody
persist-key
persist-tun

# Certificates

ca /etc/pki/CA/certs/cacert.pem
dh /etc/openvpn/dh2048.pem
cert /etc/openvpn/gateway.ohio.cc.cert
key /etc/openvpn/gateway.ohio.cc.key

# TLS Auth
tls-auth /etc/openvpn/ta.key 0
cipher AES-256-GCM
```

`topology subnet`: use this as the default `topology net30` is obsolete

`/etc/openvpn/client.conf`
```
# Connection

dev tun
client
proto udp
keepalive 10 120
remote <VPN SERVER> 1194

# Logging

log-append /var/log/openvpn.log
status /var/log/openvpn-status.log
verb 4
mute 20

# Security

user nobody
group nobody
persist-key
persist-tun

# Certificates

ca /etc/openvpn/cacert.pem
cert /etc/openvpn/branch1.ohio.cc.cert
key /etc/openvpn/branch1.ohio.cc.key

# TLS Auth
tls-auth /etc/openvpn/ta.key 1
cipher AES-256-GCM
```

- `server NETWORK MASK`: server will take the first address of the network pool and assign the rest to incoming VPN connections
- `tls-auth`: 0 for the server, 1 for the client
- `keepalive`: (ping every N seconds) (wait for response seconds)
- `verb [0-11]` : verbosity level, 0 = no logging
- `user/group [USER]`: user/group to run openvp as. 
- `persist [tun/key]`: allow OpenVPN to retain sufficient privileges to work with the network and SSL after the privilege drop.

To stat the VPN connection on both, server and client

- `openvpn /etc/openvpn/{server|client}.conf `
#### Routing

Route traffic from one internal network to another through the tunnel

```
push "route INTERNAL_NETWORK SUBNET_MASK"
```

`push "route network subnet"`: this options allows clients to access private subnets behind the server.  The private subnet must also be made OpenVPN aware. The quotes are mandatory

Restart the server and the clients. The clients will now have a new route in the routing table

```
[root@localhost openvpn]# ip route
default via 10.0.2.1 dev enp0s8 proto static metric 106
10.0.2.0/24 dev enp0s8 proto kernel scope link src 10.0.2.156 metric 106
10.8.0.1 via 10.8.0.5 dev tun0
10.8.0.5 dev tun0 proto kernel scope link src 10.8.0.6
192.168.0.0/24 via 10.8.0.5 dev tun0 <-- internal network routed through the VPN tun
```

VPN traffic needs to be forwarded from the gateway to the internal network and back. 

Add the VPN tun interface to the firewall public zone

``` bash
firewall-cmd --zone=public --add-interface=tun0 --permanent
```

Forward incoming connections on 22 from the tunnel to the internal network

``` bash
iptables -A FORWARD -i tun0 -s 10.0.2.0/24 -d 192.168.0.0/24 -p tcp --dport 22 -j ACCEPT
```

Push all traffic through the tunnel. NOT TESTED

`server`
```
push "redirect-gateway def1
```
#### Authentication

Users will authenticate with their Linux credentials to the gateway using the pam authentication plugin

`/etc/openvpn/server/.conf`
```
plugin /usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so login
```

`/etc/openvpn/client/.conf`
```
auth-user-pass
```

The client

```
[root@localhost client]# systemctl restart openvpn-client@mobile-client
🔐 Enter Auth Username: kimchen
🔐 Enter Auth Password: ******
```

`server log`
```
AUTH-PAM: BACKGROUND: received command code: 0
AUTH-PAM: BACKGROUND: USER: kimchen
AUTH-PAM: BACKGROUND: my_conv[0] query='Password: ' style=1
10.0.2.6:59463 PLUGIN_CALL: POST /usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so/PLUGIN_AUTH_USER_PASS_VERIFY status=0
10.0.2.6:59463 TLS: Username/Password authentication succeeded for username 'kimchen'
10.0.2.6:59463 [branch1.ohio.cc] Peer Connection Initiated with [AF_INET]10.0.2.6:59463
```
#### Controlling OpenVPN with systemctl

To run OpenVPN as a daemon in the background, managed by `systemd`, the service unit file must be created first. On both, server and clients

OpenVPN-related unit files in `/usr/lib/systemd/system`. 

``` bash
systemctl enable --now openvpn-server@'SERVER_CONFIG_NAME'
systemctl enable --now openvpn-client@'CLIENT_CONFIG_NAME'
```

The @ indicates a parameterized unit file. Multiple unit files can be created for the same service, calling different configuration files. This allows for multiple configurations to be created without writing multiple unit files.

How a parametrized unit file looks

```
[root@server33 openvpn]# systemctl cat openvpn@server33.example.com.service
# /usr/lib/systemd/system/openvpn@.service
[Unit]
Description=OpenVPN Robust And Highly Flexible Tunneling Application On %I
After=network.target

[Service]
Type=notify
PrivateTmp=true
ExecStart=/usr/sbin/openvpn --cd /etc/openvpn/ --config %i.conf

[Install]
WantedBy=multi-user.target
```

#### Distributing Client Configurations with .ovpn Files

Linux clients must have the **NetworkManager-openvpn-gnome** package installed to be able to import the .**ovpn** (pro)files in the GUI

The .**ovpn** file packs the information from the client conf file and certificates. Connection profile. Requires sudo privileges when executed in shell

`.ovpn structure`
```
client
dev tun
proto udp4
remote <VPN_SERVER> 1194
persist-key
persist-tun
user nobody
group nobody
verb 5
key-direction 1 -> in relation to the tls-auth. 1 for the client

<ca>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</ca>

# client.crt
<cert>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</cert>

# client.key
<key>
-----BEGIN ENCRYPTED PRIVATE KEY-----
...
-----END ENCRYPTED PRIVATE KEY-----
</key>

# ta.key - HMAC code
<tls-auth>
-----BEGIN OpenVPN Static key V1-----
...
-----END OpenVPN Static key V1-----
</tls-auth>
```

To extract the certificate section from a certificate file

``` bash
openssl x509 -in 'CERT'
```

#### Hardening OpenVPN Servers

All SSL and TLS protocols under TLS 1.2 are deprecated and should not be used. Set min TLS version to 1.2

`server and client`
```
tls-version-min 1.2
```

Use stronger data channel cipher and enforce it by disabling cipher negotiation

`server and client`
```
AES-256-GCM
ncp-disable
```

```
Outgoing Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
```

TLS 1.2

`server and client`
```
# TLS 1.2
tls-cipher TLS-ECDHE-ECDSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-RSA-
WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-ECDSA-WITH-AES-128-GCM-SHA256:
TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256
```

Elliptic Curve Diffie-Hellman Ephemeral (ECDHE) can be used instead of the Diffie-Hellman static keys (ta.key). Use `tls-groups <CURVES>` for better flexibility

`server`
```
dh none
# Explicitly set a single elliptic curve
ecdh-curve secp384r1 
# Provide a list of preferred curves for TLS exchange
tls-group CURVE1:CURVE2:...
```

List available elliptic curves

```bash
openvpn --show-curves
openssl ecparam -list_curves
```

- `opt-verify`: ( server ) checks for compatibility between server and client settings, and disconnects clients that do not match. Checks:
	`dev-type, link-mtu, tun-mtu, proto, ifconfig, comp-lzo, fragment, keydir, cipher, auth, keysize, secret, no-replay, no-iv, tls-auth, key-method, tls-server, tls-client`
- `tls-server`: on the server, works with `tls-client` on the client
- `remote-cert-tls server` : ( client ) Ensures the server's certificate is properly validated. Adds an extra layer of security by ensuring the OpenVPN client connects only to legitimate server. During handshake. Requires critical key usage extensions in the certificate. NOT TESTED!
- `remote-cert-tls client`: used in the OpenVPN server to ensure that the client certificates have the correct key usage during the handshake.  Requires critical key usage extensions in the certificate.
- `ecdh-curve secp384r1`: Explicitly set an elliptic curve
- `dh none`: Disable DH for handshake negotiation. Use ECDHE instead.
- `verify-client-cert require`
`auth sha[128|256|512]`:   bit message hash 'SHA256' for HMAC authentication

[Man-in-the-middle attacks and client-server verifications](https://openvpn.net/community-resources/how-to/#mitm)

Check if the certificate has the necessary extensions

```
X509v3 Extended Key Usage:
                TLS Web Client/Server Authentication
            X509v3 Key Usage: critical
                Digital Signature
```

`remote-cert-tls server`: The client successfully verifies the server certificate
```
Validating certificate extended key usage
++ Certificate has EKU (str) TLS Web Client Authentication, expects TLS Web Server Authentication
++ Certificate has EKU (oid) 1.3.6.1.5.5.7.3.2, expects TLS Web Server Authentication
++ Certificate has EKU (str) TLS Web Server Authentication, expects TLS Web Server Authentication
us=681809 VERIFY EKU OK
us=682124 VERIFY OK: depth=0, C=BG, ST=Sofia, L=Sofia, O=Ohio, OU=IT, CN=headoffice.ohio.cc
```

The server successfully verifies the client certificate

```
Validating certificate extended key usage
192.168.137.195:1194 ++ Certificate has EKU (str) TLS Web Client Authentication, expects TLS Web Client Authentication
192.168.137.195:1194 VERIFY EKU OK
192.168.137.195:1194 VERIFY OK: depth=0, C=BG, ST=Sofia, L=Sofia, O=Ohio, OU=IT, CN=kavarna.ohio.cc
```

Further:
Users can be disallowed from saving passwords, SELinux, chroot

#### VPN Split-Tunneling

To specify a subnet that should not be routed through the VPN, in the client profile.

```
route NETWORK MASK net_gateway
```

`port-share localhost 80`: allows requests coming in on port 1194 to be redirected to port 80 on the localhost. This only works with tcp

#### Other Configuration Directives

- `client-to-client`: allows VPN endpoints to "see" each other.
- `resolv-retry infinite`
- `ifconfig-pool-persist ipp.txt [refresh interval]` : used to persistently assign IP addresses to VPN clients.
	When a client connects, the VPN server assigns it an IP address from the predefined pool in the `server` directive. The `ipp.txt` OpenVPN manages IP-to-client mapping automatically in this file. NOT WORKING!
- `float`:  ( in server conf only ) allows clients to roam on different networks without losing connection, as long as they pass authentication tests
- `mute 20`: limits the number of repetitive log messages. Suppress the first N occurrences of the same msg
- `explicit-exit-notify 1`: ensures proper connection cleanup, helping the server recognize the client has intentionally disconnected, allowing it to manage resources more efficiently
- `nobind`: prevents the client from binding to a specific local port, allowing the system to assign a random, available port for outgoing traffic. Good for avoiding port conflicts, when running multiple VPN connections or when network conf is dynamic
- `comp-lzo`: deprecated. Compression is discouraged