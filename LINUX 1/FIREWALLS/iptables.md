More info in
Apress - Pro Linux System Administration 2nd Edition

To list current rules

``` bash
sudo iptables -L
```

```
# iptables -A FORWARD -i eth1 -o eth2 -m state --state NEW,ESTAB- LISHED,RELATED -j ACCEPT

ChatGPT

The command you provided is an `iptables` rule that adds a rule to the FORWARD chain. Let's break down the components of the command:

- `iptables`: This command is used for administering the Netfilter firewall on Linux systems.
    
- `-A FORWARD`: This option specifies that the rule should be appended to the FORWARD chain. The FORWARD chain is responsible for packets that are being routed through the system.
    
- `-i eth1 -o eth2`: These options specify the network interfaces. `-i` specifies the incoming interface (`eth1` in this case), and `-o` specifies the outgoing interface (`eth2` in this case).
    
- `-m state --state NEW,ESTABLISHED,RELATED`: This option uses the `state` module to match packets based on their connection state. In this case, it matches packets that are in the NEW, ESTABLISHED, or RELATED state.
    
- `-j ACCEPT`: This specifies the action to take if a packet matches the criteria defined by the rule. In this case, it accepts (allows) packets that match the criteria.
    

Putting it all together, the command adds a rule to the firewall that allows forwarding packets from the `eth1` interface to the `eth2` interface if they are in the NEW, ESTABLISHED, or RELATED state. This rule is commonly used in scenarios where the Linux system acts as a router or gateway between two networks (`eth1` and `eth2` in this case), allowing established and related connections to pass through while also permitting new connections.
```

List the FORWARD rules

```bash
iptables -L FORWARD
```

```
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
```

The default FORWARD policy is set to ACCEPT which means that enabling forwarding via `net.ipv4.forward=1` is enough to forward packets.


#### Saving and Restoring the Configuration

This works for [[gitea/LINUX 1/FIREWALLS/Firewalld]] as well. Both services must not be running at the same time

To dump the current configuration on screen

``` bash
iptables-save
```

To save it, we can use redirection

``` bash
iptables-save > 'firewall-backup'
```

To restore the configuration first clear the current config

``` bash
iptables -F
iptables-restore 'firewall-backup'
iptables -L
```

