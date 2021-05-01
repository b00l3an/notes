# Steel Mountain



## Recon 



### nmap

```nmap
$ sudo nmap -sC -sV -oN steelmountain 10.10.66.223
[sudo] password for boolean: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-30 13:06 IST
Nmap scan report for 10.10.66.223
Host is up (0.015s latency).
Not shown: 988 closed ports
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=steelmountain
| Not valid before: 2021-04-29T12:05:56
|_Not valid after:  2021-10-29T12:05:56
|_ssl-date: 2021-04-30T12:07:37+00:00; 0s from scanner time.
8080/tcp  open  http               HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49163/tcp open  msrpc              Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: STEELMOUNTAIN, NetBIOS user: <unknown>, NetBIOS MAC: 02:8b:e3:66:dc:45 (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-04-30T12:07:31
|_  start_date: 2021-04-30T12:05:49

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 66.49 seconds
```

The scan reveals a second http server running ``Rejetto HTTP File Server 2.3`` . A quick google search reveals an exploit for [CVE 2014-6287](https://www.exploit-db.com/exploits/39161) which allows for RCE (Remote Code Execution) .



## Foothold

Using the metasploit framework to run the exploit we quickly get a shell 



![steel-mountain-msf-rejetto](/home/boolean/notes/tryhackme/assets/steel mountain/steel-mountain-msf-rejetto.png)



This gets a shell as the user bill



## Privesc



To escalate privileges we are going to use a script called [PowerUp](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) this will run checks for any known privesc vulnerabilities.

One of the main things that this module highlights to to look out for the CanRestart:True, which is the case for the application AdvancedSystemCareService9. With this flag set to true, it tells us that we have permission to restart the application. Additionally the folder is writable. 



Combining these two facts together, we can replace the existing executable with a poisoned one and restart the service so the server will run it as LocalSystem.



![steel-mountain-privesc-check](/home/boolean/notes/tryhackme/assets/steel mountain/steel-mountain-privesc-check.png)

### Create payload

![steel-mountain-msfvenom](/home/boolean/notes/tryhackme/assets/steel mountain/steel-mountain-msfvenom.png)

