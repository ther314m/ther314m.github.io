---
title: ColddBox - Easy
date: 2021-07-15 19:45:00 +0200
categories: [Tryhackme, ColddBox]
tags: [Tryhackme, easy, CTF, ]  
comments: false
pin: true
image:
  src: https://tryhackme-images.s3.amazonaws.com/room-icons/442c3b7bab5e3f52b5341cfa9e52e5c0.png
  alt: image alternative text
---
## Tryhackme room link:
><a href="https://tryhackme.com/room/colddboxeasy">`Simple CTF`</a>

<h1 style="color: #a9db92 ;"><b>IP</b></h1>

Hogy ne felejtsük el, exportáljuk a `tryhackme`-s `IP`-t az adott terminálba.

```bash
export IP=<Tryhackme room IP>
```

<h1 style="color: #a9db92 ;"><b>ENUMERÁCIÓ</b></h1>

Futtassunk egy `nmap`-et, scanneljük le a hálózatot:

~~~bash
nmap -sCV -A -p- -Pn -vv -oA colddbox.easy $IP
~~~

Kimenet:

~~~report
[...]

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.1.31
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: ColddBox | One more machine
4512/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4e:bf:98:c0:9b:c5:36:80:8c:96:e8:96:95:65:97:3b (RSA)
|   256 88:17:f1:a8:44:f7:f8:06:2f:d3:4f:73:32:98:c7:c5 (ECDSA)
|_  256 f2:fc:6c:75:08:20:b1:b2:51:2d:94:d6:94:d7:51:4f (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

[...]
~~~

Láthatjuk, hogy egy `Apache httpd 2.4.18` webserver fut a 80-as porton és egy `OpenSSH` kiszolgáló a 4512-es porton. Ha a colddbox-os `IP`-re látogatunk, akkor láthatjuk, hogy ez egy `WordPress` site:

![image tooltip here](/assets/myimages/01.png)