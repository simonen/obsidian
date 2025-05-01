
Mounting a partition makes its contents accessible through a specific directory
#### Device Names, UUIDs or Disk Labels

UUIDs: universally unique ID. Used by default for device mounting in RHEL9

#### Persistent Mounts with fstab

man fstab
man blkid

Anatomy of the /etc/fstab file:

`<DEVICE> <MOUNTPOINT> <TYPE> <OPTIONS> <DUMP> <PASS>`

* DEVICE: device file. Can be LABEL='FILESYSTEM_LABEL' or UUID="UUID_NUMBER"
* MOUNTPOINT: directory or kernel interface the device needs to be mounted on. **none** if swap
* FILESYSTEM type to try: xfs, etx4, swap. Comma separated values can be used. udf,iso9660 will try fs for DVD and CD-ROM respectively
* MOUNT OPTIONS: mount options. defaults. Comma-separated options can be provided. 
* DUMP SUPPORT: legacy feature. 0 in most cases
* AUTOCHECK: the order in which filesystems should be checked
	* 1 if filesystem that should be autochecked at boot is root fs 
	* 2 for other filesystems. 
	* Always 0 for network filesystems. Commonly set to 0. No check

Entries in the /etc/fstab file are processed from top to bottom

SWAP is mounted on a kernel interface. KIs do not start with /

Mount options:
* defaults:  rw, suid, dev, exec, auto, nouser, and async
* noauto: do not mount when mount -a is given
* acl: access-control list
* user_xattr: user extended attributes
* ro: read-only mode
* user: allow user to mount
* owner: allow device owner to mount
* **atime / noatime**: allows, not allows file access time to be modified
* **noexec / exec**: allows, not allows programs to be executed
* error=remount-ro: remount the fs in read-only mode if a fs error occurs.

If the kernel has not registered an UUID change, the following error might occur

```
sudo mount /mnt/datax
mount: special device /dev/disk/by-uuid/ccd60fc3-bbaf-40e5-a93e-43743f9176d9
does not exist
```

Reload the udev device to redetect the UUID and correct the symbolic link /dev/disk/by-uuid

```
udevadm control --reload
```

#### Mounting a Filesystem with systemd mount units

[[LINUX/SERVICES/Systemd Units#Systemd Mount Units]]

Systemd ultimately does the mounts. It generates files off of the fstab file in
**/run/systemd/generator**

Manual mount files are made in /etc/systemd/system/\*.mount
The name of the .mount file corresponds to the directory the filesystem is mounted on
