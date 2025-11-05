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
[[Feroxbuster]] scan for two subdomains
```shell

```
m8o1UUavnBLPU5hp