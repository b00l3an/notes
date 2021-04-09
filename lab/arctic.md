# Arctic (Retired)

Retired at the time of attempt. Following the official write up and [ippsec](https://www.youtube.com/watch?v=e9lVyFH7-4o&abab)
## Recon 
### nmap
```bash
sudo nmap -sC -sV -p- -oA nmap/arctic 10.10.10.11
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-25 22:54 GMT
Nmap scan report for 10.10.10.11
Host is up (0.030s latency).
Not shown: 65532 filtered ports
PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
