# Atom 

## Recon

### nmap

```bash
(boolean㉿kalibox)[~/htb/atom]$ sudo nmap -sC -sV -p- -oN nmap/atom-allports 10.129.115.22 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-17 21:41 IST
Nmap scan report for 10.129.115.22
Host is up (0.037s latency).
Not shown: 65528 filtered ports
PORT     STATE SERVICE      VERSION
80/tcp   open  http         Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: Heed Solutions
135/tcp  open  msrpc        Microsoft Windows RPC
443/tcp  open  ssl/http     Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: Heed Solutions
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
6379/tcp open  redis        Redis key-value store
7680/tcp open  pando-pub?
Service Info: Host: ATOM; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h20m02s, deviation: 4h02m32s, median: 0s
| smb-os-discovery: 
|   OS: Windows 10 Pro 19042 (Windows 10 Pro 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: ATOM
|   NetBIOS computer name: ATOM\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-04-17T13:46:41-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-04-17T20:46:40
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 374.74 seconds
```

### ffuf

```bash
images                  [Status: 301, Size: 342, Words: 22, Lines: 10]
webalizer               [Status: 403, Size: 304, Words: 22, Lines: 10]
Images                  [Status: 301, Size: 342, Words: 22, Lines: 10]
phpmyadmin              [Status: 403, Size: 304, Words: 22, Lines: 10]
IMAGES                  [Status: 301, Size: 342, Words: 22, Lines: 10]
releases                [Status: 301, Size: 344, Words: 22, Lines: 10]
licenses                [Status: 403, Size: 423, Words: 37, Lines: 12]
server-status           [Status: 403, Size: 423, Words: 37, Lines: 12]
                        [Status: 200, Size: 7581, Words: 2135, Lines: 192]
con                     [Status: 403, Size: 304, Words: 22, Lines: 10]
aux                     [Status: 403, Size: 304, Words: 22, Lines: 10]
Releases                [Status: 301, Size: 344, Words: 22, Lines: 10]
prn                     [Status: 403, Size: 304, Words: 22, Lines: 10]
server-info             [Status: 403, Size: 423, Words: 37, Lines: 12]
```

### gobuster - vhost

Zero results



## Contacts

MrR3boot@atom.htb

