# CyberHeroes  
Our **hero** is looking for a way to log into a site  
First we will find what services the **_TARGET_** host is offering  
```sh
# Nmap 7.92 scan initiated Sat Aug  6 21:40:15 2022 as: nmap -Pn -sV -sC -T4 -oA scans/<target>.init <target ip>
Nmap scan report for <target ip>
Host is up (0.26s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 a6:c4:92:1a:b9:cf:de:af:ae:c8:3d:1d:ca:b3:a2:5c (RSA)
|   256 5f:11:b2:f8:51:f3:c7:e1:11:45:62:bd:1b:f4:17:f8 (ECDSA)
|_  256 08:75:1a:1f:01:d2:9b:0e:eb:0a:c0:7f:e5:31:6e:4d (ED25519)
80/tcp open  http    Apache httpd 2.4.48 ((Ubuntu))
|_http-title: CyberHeros : Index
|_http-server-header: Apache/2.4.48 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Aug  6 21:40:36 2022 -- 1 IP address (1 host up) scanned in 20.77 seconds
```
Parsing the index page of the webserver on port 80 we find a link to *login.html* in a sidebar  
```sh
curl -is http://$TARGET
...
HTTP/1.1 200 OK                                                                                   
Date: Sun, 07 Aug 2022 01:41:36 GMT                                                               
Server: Apache/2.4.48 (Ubuntu)                                                                    
Last-Modified: Wed, 02 Mar 2022 19:07:59 GMT                                                      
ETag: "19a8-5d940fff601c0"                                                                        
Accept-Ranges: bytes                                                                              
Content-Length: 6568                                                                              
Vary: Accept-Encoding                                                                             
Content-Type: text/html
...
<nav id="navbar" class="nav-menu navbar">                                                   
        <ul>                                                                                      
          <li><a href="#hero" class="nav-link scrollto active"><i class="bx bx-home"></i> <span>Ho
me</span></a></li>
          <li><a href="#about" class="nav-link scrollto"><i class="bx bx-user"></i> <span>About</s
pan></a></li>                                                                                     
          <li><a href="login.html" class="nav-link scrollto"><i class="bx bx-server"></i> <span>Lo
gin</span></a></li>
        </ul>                                                                                     
      </nav>
```
Parsing the source for that page we find some inline javascript  
```js
curl -is http://$TARGET/login.html
...
<script>
    function authenticate() {
      a = document.getElementById('uname')                                                        
      b = document.getElementById('pass')                                                         
      const RevereString = str => [...str].reverse().join('');
      if (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS")) {
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function() {
          if (this.readyState == 4 && this.status == 200) {
            document.getElementById("flag").innerHTML = this.responseText ;
            document.getElementById("todel").innerHTML = "";
            document.getElementById("rm").remove() ;
          }
        };            
        xhttp.open("GET", "RandomLo0o0o0o0o0o0o0o0o0o0gpath12345_Flag_"+a.value+"_"+b.value+".txt"
, true);                                                                                          
        xhttp.send();                                                                             
      }                                          
      else {
        alert("Incorrect Password, try again.. you got this hacker !")
      }
    }
  </script>
...
```
I see a hardcoded publicly exposed username/password combo  
This inline script contains a function named authenticate that grabs a username  
and a password from a form on the page.
It then compares them to a hardcoded value and  
a local variable defined as the reverse of a string value.  
- username refers to variable a --> a = document.getElementById('uname')  
- password refers to variable b --> b = document.getElementById('pass')  
- const RevereString = str => [...str].reverse().join('');  
A boolean comparison is made (the if statement)  
to determine the page returned to the browser. Or rather an event not a page  
is returned. That event either reveals the flag or admonishes the user to "try again.."  
This challenge demonstrates one of a large variety of reasons why client side JS  
is no place to put authentication code.
