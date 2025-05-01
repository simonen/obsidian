
To ping IP all addresses in a subnet
\# nmap -sn *NETWORK/SUBNET_MASK*

```
[kimchen@dnsserver ~]$ nmap -sn 192.168.137.0/24
Starting Nmap 7.92 ( https://nmap.org ) at 2024-03-26 09:16 EET
Nmap scan report for server15 (192.168.137.12)
Host is up (0.012s latency).
Nmap scan report for dnsserver (192.168.137.23)
Host is up (0.00056s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 3.53 seconds
```
**-sn**: no port scan

To add a blank line between entries
\# **nmap -sn** *192.168.137.0/24* | **awk '/Nmap/{print ""}1'**

To collect a list of all hosts and their IP addresses
\# nmap -sn 192.168.137.0/24 | grep 'Nmap scan report for' | cut -d " " -f 5,6 -> python: string.split(" ")\[4:6]

```
192.168.137.1
server15 (192.168.137.12)
dnsserver (192.168.137.23)
```

Probe for open ports
\# **nmap -sS** *TARGET*
-**sS**: TCP SYN scan. Fast and good.

To probe a LAN for open TCP (default) ports 
\# **nmap --open** *NETWORK/SUBNET_MASK*

To scan for TCP and UDP open ports
\# **nmap -sU -sT** *NETWORK/SUBNET_MASK*

To scan for specific open UDP ports
\# **nmap -sU** *UDP1,UDP2,UDP3 NETWORK/SUBNET_MASK*

