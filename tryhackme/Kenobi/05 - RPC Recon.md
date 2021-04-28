# nmap

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
