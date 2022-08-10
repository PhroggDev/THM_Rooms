# IDE  
![ide logo](/ide/imgs/ide_room_logo.png)

IDE, an easy box to polish your enumeration skills!

## Summary
  * Start Machine::
  * Find open ports using nmap
  * Access ports discovered
  * Where possible determine version of server software
  * Seek out publicly released known exploits
  * Once local access is achieved, escalate privileges to find all the flags

### 1. Nmap enumeration

I used an [**Aggressive**](https://nmap.org/book/vscan-examples.html) scan (the `-A` option), hoping to see slightly more verbose version detection. The `-T4` option can be used on the TryHackMe platform since the machines we attack are on a virtual private network using [private addressing space](https://en.wikipedia.org/wiki/Private_network#Private_IPv4_addresses). Tantamount to being on a LAN. In the field I would most likely reduce that to `-T3` or `-T2` to reduce the chance of being detected by an IDS or worse overwhelming a production server, spoiling my probe results. Before scanning I added an entry in `/etc/hosts` of my attack box to point my local resolver at the ip address I learned after the target box was started.

```console
# Nmap 7.92 scan initiated Sat Nov  6 22:10:06 2021 as: nmap -A -T4 -oA scans/nmap_init ide.thm
Nmap scan report for ide.thm (10.10.64.67)
Host is up (0.23s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to <my attacker IP>
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e2:be:d3:3c:e8:76:81:ef:47:7e:d0:43:d4:28:14:28 (RSA)
|   256 a8:82:e9:61:e4:bb:61:af:9f:3a:19:3b:64:bc:de:87 (ECDSA)
|_  256 24:46:75:a7:63:39:b6:3c:e9:f1:fc:a4:13:51:63:20 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)

Network Distance: 4 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT       ADDRESS
1   106.56 ms 10.13.0.1
2   ... 3
4   241.81 ms ide.thm (10.10.64.67)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Nov  6 22:10:48 2021 -- 1 IP address (1 host up) scanned in 42.41 seconds
```

I see some standard services:  
* FTP being served by vsFTPd 3.0.3 --> *always good to see version strings when we probe*
* SSH being served up by OpenSSH 7.6p1 --> *Not the most up to date or secure, but not grossly insecure*
* HTTP served up by Apache2 on Ubuntu --> *This turns out to be just the first httpd daemon running on this box*

The first two, FTP and SSH, are relatively stable and secure servers. Further probing by loading up the Apache homepage in a browser gives the appearance this is a distro specific  standard install. Hinted at by `http-title: Apache2 Ubuntu Default Page: It works`

![Default Page](/ide/imgs/screenshot_default_page.png)

In the rush to hack over a web port I neglected a feature of the ftp server `Anonymous FTP login allowed (FTP code 230)` that Nmap revealed. Let's fire up an ftp client and see what's publicly accessible. You never know what secrets got left out in the open.

#### Security through obscurity is **NO SECURITY** at all

A quick bout of Command Line Commando `ftp ide.thm` using credentials of username--> anonymous and leaving password blank drops us in the ftp directory. There we find a directory named `...`
```sh
cd ...
ftp> ls -lA
-rw-r--r--  1 0   0   151 Jun 18 06:11  -
```  
Now that's curious. Inside that oddly named directory is a file named `-` *("dash")*  
Let's grab it and see what's inside  
```txt
Hey john,
I have reset the password as you have asked. Please use the default password to login.
Also, please take care of the image file ;)
- drac.
```  
So, we'll keep this in our back pocket for now. A password got reset and instructions to use a **default password** to login somewhere. We also have two prospective users `john` and `drac` to tuck away inside a `user.txt`.

The topic of this room is "Enumeration" so let's look at the ports above the so called **privileged ports** aka higher than 1024  
```nmap
# Nmap 7.92 scan initiated Sat Nov  6 22:22:23 2021 as: nmap -sV -T4 -p 1024- -oN scans/nmap_above1024 ide.thm
Nmap scan report for ide.thm (10.10.64.67)
Host is up (0.24s latency).
Not shown: 64511 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
62337/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Nov  6 22:31:06 2021 -- 1 IP address (1 host up) scanned in 523.21 seconds
```  
Loading this new url in our browser redirects us to a login page for [Codiad Web IDE](http://codiad.com). From a link to the [Github Repo](https://github.com/Codiad/Codiad) we see it is a web based IDE for code development that has made note it is no longer maintained.  Last commit was *Mar 21, 2020*.  
Capturing a login can be done with a proxy or easier still, using the dev tools in either Firefox or Chrome. In Firefox, what I prefer to use, open up the Network tab in DevTools before logging in so that the exchange is captured in the tool. Log in with credentials we can spot in the captured **POST** request that we'll save to a file for reference in the next step.  Click on the **POST** request in the Network tab, Click on Copy--->Copy Request Headers, paste that into a file. Then Copy-->Copy Post Data and paste that below the headers.  
```text
POST http://ide.thm:62337/components/user/controller.php?action=authenticate HTTP/1.1
Host: ide.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5 --compressed
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Origin: http://ide.thm:62337
Connection: keep-alive
Referer: http://ide.thm:62337/
Cookie: f9c7294bc8f6035df784b56b800b122c=m9k8u4546g47no5uerg3kvjtr3

username=testuser&password=testpass&theme=default&language=en
```  
btw that's what our request looked like on the wire, so to speak, when it got sent over to Apache2.  We know the **Response** returned **JSON** from the headers returned including `Content-Type: application/json`. We now have all we need to attempt to brute-force a password on the Codiad Server on Port 62337

### 2. Hydra Password Enumeration

[Hydra](https://github.com/vanhauser-thc/thc-hydra) is straight forward brute-forcing tool. Tell it the user or users and the password or list of passwords. Beyond that just give it an **IP** and protocol **method** and we're off to the races. For this Codiad server I will include a port number since **http** is usually served on port **80**.  
Command looks something like this  
`hydra -L <PATH TO USERLIST> -P <PATH TO PASSWORDS LIST> protocol-method://<$IP>$URL:<POST DATA>:<FAIL TEXT>`  
Looking for a quick easy win we'll use `probable-v2-top207.txt` from [Seclist](https://github.com/danielmiessler/SecLists) add an `-s $PORT#` since Apache2 is running on a non-standard port. Cram our POST DATA and Error Message in and off we go...
```console
hydra -L user.txt -P /usr/share/seclists/Passwords/probable-v2-top207.txt -s 62337 "http-post-form://ide.thm/components/user/controller.php?action=authenticate:username=^USER^&password=^PASS^&theme=default&language=en:Incorrect Username"
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-11-08 14:52:29
[DATA] max 16 tasks per 1 server, overall 16 tasks, 1035 login tries (l:5/p:207), ~65 tries per task
[DATA] attacking http-post-form://10.10.155.168:62337/components/user/controller.php?action=authenticate:username=^USER^&password=^PASS^&theme=default&language=en:Incorrect Username
[62337][http-post-form] host: 10.10.155.168   login: john   password: xxxxxxxx
```  
A few seconds later and we have valid credentials. User *`drac`* did reset the user *`john`* to a **well-known** default password.

### 3. Exploit and get Local Shell Access  
  * Find exploit using *searchsploit* on **Kali**
  * Edit the *sploit* to use our attacking host
  * Setup 3 terminals to implement the *sploit* chosen
  * Get **shell** , Escalate privilege using found credentials
  * Escalate to **SuperUser** and enjoy the ***PWNing***

  {To be continued...}
