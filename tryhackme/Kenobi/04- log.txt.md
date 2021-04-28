# Information gathered from log.txt

## SSH Information
```bash
$ cat log.txt          
Generating public/private rsa key pair.                                                                  
Enter file in which to save the key (/home/kenobi/.ssh/id_rsa):                                          
Created directory '/home/kenobi/.ssh'.                                                                   
Enter passphrase (empty for no passphrase): 
Enter same passphrase again:                                                                             
Your identification has been saved in /home/kenobi/.ssh/id_rsa.                                                                                                                                                    
Your public key has been saved in /home/kenobi/.ssh/id_rsa.pub.
The key fingerprint is:                                                                                  
SHA256:C17GWSl/v7KlUZrOwWxSyk+F7gYhVzsbfqkCIkr2d7Q kenobi@kenobi     
The key's randomart image is:                                                                            
+---[RSA 2048]----+                                 
|                 |                                                                                      
|           ..    |                                                                                      
|        . o. .   |                                                                                      
|       ..=o +.   |                                                                                      
|      . So.o++o. |                
|  o ...+oo.Bo*o  |                                                                                      
| o o ..o.o+.@oo  |                                                                                      
|  . . . E .O+= . |                                                                                      
|     . .   oBo.  |                          
+----[SHA256]-----+                                     
```

## ftp Information
```bash
# This is a basic ProFTPD configuration file (rename it to 
# 'proftpd.conf' for actual use.  It establishes a single server
# and a single anonymous login.  It assumes that you have a user/group
# "nobody" and "ftp" for normal operation and anon.

ServerName                      "ProFTPD Default Installation"                                           
ServerType                      standalone                                                               
DefaultServer                   on                                                                       
# Port 21 is the standard FTP port.                 
Port                            21                  
                                                                                 
# Don't use IPv6 support by default.
UseIPv6                         off

# Umask 022 is a good standard umask to prevent new dirs and files
# from being group and world writable.
Umask                           022

# To prevent DoS attacks, set the maximum number of child processes
# to 30.  If you need to allow more than 30 concurrent connections
# at once, simply increase this value.  Note that this ONLY works
# in standalone mode, in inetd mode you should use an inetd server
# that allows you to limit maximum number of processes per service
# (such as xinetd).
MaxInstances                    30

# Set the user and group under which the server will run.
User                            kenobi
Group                           kenobi

# To cause every FTP user to be "jailed" (chrooted) into their home
# directory, uncomment this line.
#DefaultRoot ~

# Normally, we want files to be overwriteable.
AllowOverwrite          on

# Bar use of SITE CHMOD by default
<Limit SITE_CHMOD>
  DenyAll
</Limit>

# A basic anonymous configuration, no upload directories.  If you do not
# want anonymous users, simply delete this entire <Anonymous> section.
<Anonymous ~ftp>
  User                          ftp
  Group                         ftp

  # We want clients to be able to login with "anonymous" as well as "ftp"
  UserAlias                     anonymous ftp

  # Limit the maximum number of anonymous logins
  MaxClients                    10

  # We want 'welcome.msg' displayed at login, and '.message' displayed
  # in each newly chdired directory.
  DisplayLogin                  welcome.msg
  DisplayChdir                  .message

  # Limit WRITE everywhere in the anonymous chroot
  <Limit WRITE>
    DenyAll
  </Limit>
</Anonymous>
```

