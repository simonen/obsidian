
Provides remote connection to a server running **sshd daemon**, using an ssh client

**man sshd**

config file
**/etc/ssh/ssh_config**
**/etc/ssh/sshd_config**

**host keys**: authenticate hosts
**public keys**: authenticate users

Transmissions are encrypted with the public key and decrypted with the private key

Test the **ssh_config** for syntax errors

``` bash
sshd -t
```

To display the active SSH sessions on a host

``` bash
echo $SSH_CLIENT
```

```
192.168.137.1 49926 22
CLIENT_IP CLIENT_PORT SERVER_PORT
```

remote servers' public keys are stored in `~/.ssh/known_hosts`

#### Customizing BASH Prompt for SSH

Customize the prompt so that it is easier to acknowledge ssh sessions
In .**bashrc**

```
if [ -n "$SSH_CLIENT" ]; then text=" ssh"
fi
export PS1='\[\e[0;36m\]\u@\h:\w${text}$\[\e[0m\] '
```

The prompt's color on the **ssh** server has now changed, thus indicating ssh session more clearly

if \[ -n "$SSH_CLIENT" ]; then text = VALUE : if the $SSH_CLIENT does not return an output, thus no open ssh sessions, create a variable **text** with value **ssh**

text=" *TEXT* " 
kimchen@backupserver:~ *TEXT* $

\[\e\[0;31m\]: the code block that determines color
\[\e\[0m\]: turns off custom colors for commands and command output

#### Listing Supported Encryption Algorithms

**man ssh**
The supported encryption algorithms can be found under the -Q options

To list supported key types:

``` bash
ssh -Q key
```

To list supported ciphers

``` bash
ssh -Q cipher
```

#### Using Graphical Applications in SSH environment

The remote host must have an **X** server running to allow graphical apps to be displayed over SSH
The remote host must be allowed to display screens on the local host

**-Y** allows the remote host to display screens
**$ ssh -Y** *username@remotehost *

**ForwardX11 yes**  in the config file enables remote graphical display by default
#### SCP - Securely Copying Files 

``` bash
scp "SOURCE" "DESTINATION"
```

**Examples:** 

Copy (pull) a file from a remote host to local computer home dir

``` bash
scp "REMOTE_SOURCE" "LOCAL_DESTINATION"
```

Copy (push) a file from local computer to remote server

``` bash
scp "LOCAL_SOURCE" "REMOTE_DESTINATION"
```

`-r`: copies the entire subdirectory structure
`-P`: (uppercase) to specify non-default ssh port

#### Mounting Remote Filesystems with SSHFS

Install the **sshfs** package. Also installs **FUSE**: Filesystem in Userspace

The remote filesystems is mounted in a local directory with write permissions

``` bash
sshfs "USER@REMOTEHOST":"DIR" "/MOUNTPOINT"
```

To list **sshfs** mounts

``` bash
findmnt | grep sshfs
```

```
└─/home/kimchen/remotefs              kimchen@server23.example.com: fuse.sshfs rw,nosuid,nodev,relatime,user_id=1000,group_id=1000
```

Options
**-o reconnect:** tells sshfs to reconnect if the connection is interrupted 

To safely unmount an SSH mounted fs

``` bash
fusermount -u "MOUNTPOINT"
```

#### SFTP - Securely Transferring Files

Secure FTP

Uses the FTP protocol to securely transfer file.
**put** - upload
**get** - download
**sshd** service is required to be running on the remote host

Current working directory is used to download to and upload from files

**$ sftp *user@remotehost*
sftp>** 

**lpwd** - local current working directory
**lcd** - change local working directory
**pwd** - print working directory on the remote host
**lls** - list local directory contents

**$ put / get** *source destination*