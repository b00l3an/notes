# SMB Enumeration - nmap

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

# SMB Enumeration - smbmap
```bash
$ smbmap -H 10.10.45.110
[+] Guest session       IP: 10.10.45.110:445    Name: 10.10.45.110                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        anonymous                                               READ ONLY
        IPC$                                                    NO ACCESS       IPC Service (kenobi server (Samba, Ubuntu))
```

# Retrieving file from Share
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