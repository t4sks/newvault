we have ![[Pasted image 20251021130511.png]]
this questions and start from machine scan with nmap
results of scaning:
```shell
┌──(user㉿kali)-[~]
└─$ nmap -A -T4 -p1-1000 10.10.113.235
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-21 13:08 +07
Nmap scan report for 10.10.113.235
Host is up (0.11s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9f:1d:2c:9d:6c:a4:0e:46:40:50:6f:ed:cf:1c:f3:8c (RSA)
|   256 63:73:27:c7:61:04:25:6a:08:70:7a:36:b2:f2:84:0d (ECDSA)
|_  256 b6:4e:d2:9c:37:85:d6:76:53:e8:c4:e0:48:1c:ae:6c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Wavefire
|_http-server-header: Apache/2.4.29 (Ubuntu)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT       ADDRESS
1   115.70 ms 10.11.0.1
2   116.87 ms 10.10.113.235

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.72 seconds
```
we know that open 2 ports 22 and 80, lets check `http://10.10.113.235` ![[Pasted image 20251021131422.png]]
firs domen what i saw it's in mail mafialive.thm, add in `/etc/hosts`
![[Pasted image 20251021132655.png]]
next lets check this endpoint with feroxbuster 
```shell
feroxbuster -u http://mafialive.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt,bak,zip,tar,conf,inc -t 200 -s 200,204,301,302,403
```
results:
```shell

```
we find endpoints `robots.txt` `test.php` 
`robots.txt`:
![[Pasted image 20251021180716.png]]
we find endpoint `test.php agian`
![[Pasted image 20251021133706.png]]
we have button and nothing interesting in html or another data, lets click
![[Pasted image 20251021134016.png]]
we find new endpoint with strange way, must explorer more because "Control is an ilussion"
try ti get a cod of mrrobots.php
```URL

http://

```

