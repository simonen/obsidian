
https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/

In linux network interfaces names are based on firmware, device topology, device type.

Devices get their names from the biosdevname udev helper utility

Devices can use BIOS name example:
`em[1,2..N]`: embedded card 
`p4p1`: PCI slot4, port 1
`p2p1_1`: Virtual interface p\<slot>p\<port>

biosdevname can be enabled by adding it as a boot option:
biosdevname=0 - enabled, 1 - disabled

`en`: for ethernet interfaces start
`eno987` - onboard, ethernet device with unique index number 987
`ens1`: PCI slot index number
`enp0s8`: physical location of the connector
`enx7f291992`: combining the MAC address

`wl`: for wlan interfaces begin
``ww``: for wwan interfaces

type of adapters:
o: onboard adapter
s: hotplug slot
p: PCI location
x: created by adminstrators for a device name based on MAC

If fixed name cannot be determined, `eth0` is used




