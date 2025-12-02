Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-02 10:06 GMT
Warning: 10.10.11.97 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.11.97
Host is up (0.10s latency).
Not shown: 65314 closed tcp ports (reset), 219 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.52
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: Host: gavel.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 976.11 seconds

feroxbuster -u http://gavel.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt,bak,zip,tar,conf,inc -t 200 -s 200,204,301,302,403


───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://gavel.htb
 🚀  Threads               │ 200
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
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
403      GET        9l       28w      274c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter


---
200      GET       78l      213w     4281c http://gavel.htb/login.php
301      GET        9l       28w      309c http://gavel.htb/includes => http://gavel.htb/includes/
200      GET       84l      301w     4485c http://gavel.htb/register.php
200      GET      222l     1037w    13984c http://gavel.htb/index.php
200      GET        7l     1030w    84378c http://gavel.htb/assets/vendor/bootstrap/js/bootstrap.bundle.min.js
200      GET        2l     1294w    89501c http://gavel.htb/assets/vendor/jquery/jquery.min.js
200      GET       58l      108w     1274c http://gavel.htb/assets/vendor/fontawesome-free/package.json
200      GET      266l     1319w   187333c http://gavel.htb/assets/img/certificate.jpg
200      GET      324l     2559w   196986c http://gavel.htb/assets/img/scroll.jpg
200      GET    11299l    22157w   212581c http://gavel.htb/assets/css/sb-admin-2.css
301      GET        9l       28w      307c http://gavel.htb/assets => http://gavel.htb/assets/
302      GET        0l        0w        0c http://gavel.htb/admin.php => index.php
200      GET      102l      397w     3798c http://gavel.htb/assets/items.json
200      GET        7l       33w     1265c http://gavel.htb/assets/js/sb-admin-2.min.js
302      GET        0l        0w        0c http://gavel.htb/logout.php => index.php
200      GET     4422l    25758w  1976010c http://gavel.htb/assets/img/welcome.png
200      GET      222l     1035w    14008c http://gavel.htb/
302      GET        0l        0w        0c http://gavel.htb/inventory.php => index.php
301      GET        9l       28w      306c http://gavel.htb/rules => http://gavel.htb/rules/
200      GET        0l        0w        0c http://gavel.htb/includes/config.php
200      GET        0l        0w        0c http://gavel.htb/includes/auction.php
200      GET        0l        0w        0c http://gavel.htb/includes/session.php

[####################] - 5m    270000/270000  823/s   http://gavel.htb/includes/ 
[####################] - 5m    270000/270000  822/s   http://gavel.htb/assets/vendor/jquery/ 
[####################] - 7s    270000/270000  37948/s http://gavel.htb/assets/vendor/fontawesome-free/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 0s    270000/270000  1356784/s http://gavel.htb/assets/js/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 7s    270000/270000  38012/s http://gavel.htb/assets/vendor/jquery-easing/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 5m    270000/270000  871/s   http://gavel.htb/rules/