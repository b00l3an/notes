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

