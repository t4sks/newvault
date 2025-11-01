nmap scan and feroxbuster:
```shell
nmap -A -T4 -p- 10.10.217.155
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-01 08:06 GMT
Nmap scan report for 10.10.217.155
Host is up (0.11s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 38:91:12:c7:ba:3c:78:f7:9a:7f:0c:10:1c:53:90:df (RSA)
|   256 41:da:26:12:cd:e2:6e:49:7f:be:6b:c4:6c:f5:60:52 (ECDSA)
|_  256 4d:e4:f8:4b:e0:94:b7:d1:d8:57:67:77:b0:6c:61:b0 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://creative.thm
|_http-server-header: nginx/1.18.0 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Linux 4.X|2.6.X|3.X|5.X (96%)
OS CPE: cpe:/o:linux:linux_kernel:4.15 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:5
Aggressive OS guesses: Linux 4.15 (96%), Linux 2.6.32 - 3.13 (90%), Linux 3.10 - 4.11 (90%), Linux 3.2 - 4.14 (90%), Linux 4.15 - 5.19 (90%), Linux 5.0 - 5.14 (90%), Linux 2.6.32 - 3.10 (90%), Linux 5.4 (88%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   113.10 ms 10.11.0.1
2   113.13 ms 10.10.217.155

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 137.49 seconds


feroxbuster -u http://10.10.217.155 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt,bak,zip,tar,conf,inc -t 200 -s 200,204,301,302,403
                                                                                                        
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.217.155
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
301      GET        7l       12w      178c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
[####################] - 3m    270000/270000  0s      found:0       errors:1      
[####################] - 3m    270000/270000  1684/s  http://10.10.217.155
```

