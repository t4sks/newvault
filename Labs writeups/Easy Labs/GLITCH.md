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

```
results:
```shell
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
in caido we can see more, `vulnerability_exploited [object global]` ask chatgpt and its Node.js + Express and eval() we have JavaScript RCE on server, lets take a shell from js
