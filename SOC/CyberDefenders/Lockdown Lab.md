Для анализа дампов используем volatility

Для нахождения базового адреса ядра в дампе 

```
python3 vol.py -f /media/sf_for_kali/memdump.mem windows.info
```

в результате получаем: 

```
┌──(venv)─(user㉿kali)-[~/Desktop/volatility3]
└─$ python3 vol.py -f /media/sf_for_kali/memdump.mem windows.info
Volatility 3 Framework 2.28.2
Progress:  100.00               PDB scanning finished                                                                                              
Variable        Value

Kernel Base     0xf80079213000
DTB     0x1aa000
Symbols file:///home/user/Desktop/volatility3/volatility3/symbols/windows/ntkrnlmp.pdb/EF9A48AFA50FF07C616585BB01919536-1.json.xz
Is64Bit True
IsPAE   False
layer_name      0 WindowsIntel32e
memory_layer    1 FileLayer
KdVersionBlock  0xf80079613f10
Major/Minor     15.17763
MachineType     34404
KeNumberProcessors      4
SystemTime      2024-09-10 06:14:13+00:00
NtSystemRoot    C:\Windows
NtProductType   NtProductServer
NtMajorVersion  10
NtMinorVersion  0
PE MajorOperatingSystemVersion  10
PE MinorOperatingSystemVersion  0
PE Machine      34404
PE TimeDateStamp        Sun Nov 10 07:20:39 2075
```

далее спрашивают полный путь до бинарника который связан с persistence из дампа можем извлечь дерево запущенных процессов

```
┌──(venv)─(user㉿kali)-[~/Desktop/volatility3]
└─$ python3 vol.py -f /media/sf_for_kali/memdump.mem windows.pstree
Volatility 3 Framework 2.28.2
Progress:  100.00               PDB scanning finished                        
PID     PPID    ImageFileName   Offset(V)       Threads Handles SessionId       Wow64   CreateTime      ExitTime        Audit   Cmd     Path

4       0       System  0xce0652a67200  125     -       N/A     False   2024-09-10 05:28:40.000000 UTC  N/A     -       -       -
* 104   4       Registry        0xce0652ba8040  4       -       N/A     False   2024-09-10 05:28:33.000000 UTC  N/A     Registry        -       -
* 300   4       smss.exe        0xce06540a10c0  2       -       N/A     False   2024-09-10 05:28:40.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\smss.exe  \SystemRoot\System32\smss.exe   \SystemRoot\System32\smss.exe
408     396     csrss.exe       0xce0654cb70c0  11      -       0       False   2024-09-10 05:29:02.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\csrss.exe %SystemRoot%\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,20480,768 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=sxssrv,4 ProfileControl=Off MaxRequestThreads=16       C:\Windows\system32\csrss.exe
484     476     csrss.exe       0xce0654e7d140  10      -       1       False   2024-09-10 05:29:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\csrss.exe %SystemRoot%\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,20480,768 Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServerDllInitialization,3 ServerDll=sxssrv,4 ProfileControl=Off MaxRequestThreads=16       C:\Windows\system32\csrss.exe
508     396     wininit.exe     0xce0654e9a080  1       -       0       False   2024-09-10 05:29:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\wininit.exe       wininit.exe     C:\Windows\system32\wininit.exe
* 640   508     lsass.exe       0xce0654ed8080  7       -       0       False   2024-09-10 05:29:07.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\lsass.exe C:\Windows\system32\lsass.exe   C:\Windows\system32\lsass.exe
* 628   508     services.exe    0xce0654eba080  7       -       0       False   2024-09-10 05:29:06.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\services.exe      C:\Windows\system32\services.exe        C:\Windows\system32\services.exe
** 768  628     svchost.exe     0xce0654f75080  1       -       0       False   2024-09-10 05:29:12.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k DcomLaunch -p -s PlugPlay    C:\Windows\system32\svchost.exe
** 2304 628     svchost.exe     0xce0654d4e0c0  16      -       0       False   2024-09-10 05:30:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s Winmgmt        C:\Windows\system32\svchost.exe
** 2180 628     svchost.exe     0xce0656ddd080  3       -       1       False   2024-09-10 05:31:50.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s CDPUserSvc       C:\Windows\system32\svchost.exe
** 776  628     svchost.exe     0xce0657050080  3       -       0       False   2024-09-10 05:29:50.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k NetworkServiceNetworkRestricted -p -s PolicyAgent    C:\Windows\system32\svchost.exe
** 2440 628     svchost.exe     0xce06571a3300  4       -       0       False   2024-09-10 05:30:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalService -s W32Time      C:\Windows\system32\svchost.exe
** 908  628     svchost.exe     0xce0654fa4080  13      -       0       False   2024-09-10 05:29:14.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k RPCSS -p     C:\Windows\system32\svchost.exe
** 1292 628     svchost.exe     0xce0656de0080  3       -       0       False   2024-09-10 05:29:27.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s gpsvc  C:\Windows\system32\svchost.exe
** 780  628     svchost.exe     0xce0657052080  5       -       0       False   2024-09-10 05:29:50.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s IKEEXT C:\Windows\system32\svchost.exe
** 1680 628     svchost.exe     0xce0656eb4080  4       -       0       False   2024-09-10 05:29:34.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalService -p -s FontCache C:\Windows\system32\svchost.exe
** 1808 628     svchost.exe     0xce0656f94080  5       -       0       False   2024-09-10 05:29:38.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k NetworkService -p -s LanmanWorkstation       C:\Windows\System32\svchost.exe
** 3856 628     svchost.exe     0xce0657309080  7       -       0       False   2024-09-10 05:32:27.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalService -p -s CDPSvc    C:\Windows\system32\svchost.exe
** 2064 628     svchost.exe     0xce06578ed080  6       -       0       False   2024-09-10 05:44:23.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s StorSvc   C:\Windows\System32\svchost.exe
** 660  628     svchost.exe     0xce0656cc30c0  5       -       0       False   2024-09-10 05:29:22.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s NcbService        C:\Windows\System32\svchost.exe
** 2452 628     svchost.exe     0xce06571cb280  15      -       0       False   2024-09-10 05:30:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k iissvcs      C:\Windows\system32\svchost.exe
*** 4332        2452    w3wp.exe        0xce06574ca080  0       -       0       False   2024-09-10 05:44:45.000000 UTC  2024-09-10 06:10:48.000000 UTC     \Device\HarddiskVolume1\Windows\System32\inetsrv\w3wp.exe       -       -
**** 900        4332    updatenow.exe   0xce0657ddb1c0  3       -       0       True    2024-09-10 06:08:23.000000 UTC  N/A     \Device\HarddiskVolume1\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\updatenow.exe    "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\updatenow.exe"       C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\updatenow.exe
** 792  628     svchost.exe     0xce0654f69080  14      -       0       False   2024-09-10 05:29:12.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k DcomLaunch -p        C:\Windows\system32\svchost.exe
*** 2696        792     SearchUI.exe    0xce065730d080  14      -       1       False   2024-09-10 05:33:42.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\SystemApps\Microsoft.Windows.Cortana_cw5n1h2txyewy\SearchUI.exe    "C:\Windows\SystemApps\Microsoft.Windows.Cortana_cw5n1h2txyewy\SearchUI.exe" -ServerName:CortanaUI.AppXa50dqqa5gqv4a428c9y1jjw7m3btvepj.mca        C:\Windows\SystemApps\Microsoft.Windows.Cortana_cw5n1h2txyewy\SearchUI.exe
*** 1704        792     RuntimeBroker.  0xce06579ad080  1       -       1       False   2024-09-10 05:34:01.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\RuntimeBroker.exe C:\Windows\System32\RuntimeBroker.exe -Embedding        C:\Windows\System32\RuntimeBroker.exe
*** 4092        792     ShellExperienc  0xce065772b080  13      -       1       False   2024-09-10 05:33:35.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe       "C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe" -ServerName:App.AppXtk181tbxbce2qsex02s8tw7hfxa9xb3t.mca C:\Windows\SystemApps\ShellExperienceHost_cw5n1h2txyewy\ShellExperienceHost.exe
*** 2036        792     RuntimeBroker.  0xce0656dd40c0  3       -       1       False   2024-09-10 05:33:46.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\RuntimeBroker.exe C:\Windows\System32\RuntimeBroker.exe -Embedding        C:\Windows\System32\RuntimeBroker.exe
** 2072 628     svchost.exe     0xce065703e080  8       -       0       False   2024-09-10 05:29:52.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s UserManager    C:\Windows\system32\svchost.exe
*** 2172        2072    sihost.exe      0xce06571d5080  5       -       1       False   2024-09-10 05:31:50.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\sihost.exe        sihost.exe      C:\Windows\system32\sihost.exe
** 1308 628     svchost.exe     0xce0656ddc080  3       -       0       False   2024-09-10 05:29:27.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k netsvcs -p -s Themes C:\Windows\System32\svchost.exe
** 412  628     svchost.exe     0xce065791a080  1       -       0       False   2024-09-10 05:33:32.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s Appinfo        C:\Windows\system32\svchost.exe
** 1052 628     svchost.exe     0xce0657783080  7       -       0       False   2024-09-10 05:34:00.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s UsoSvc C:\Windows\system32\svchost.exe
** 1564 628     svchost.exe     0xce0657a4f080  2       -       0       False   2024-09-10 06:12:41.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s WdiSystemHost     C:\Windows\System32\svchost.exe
** 1440 628     svchost.exe     0xce0656e3e080  7       -       0       False   2024-09-10 05:29:29.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted -p -s Dhcp     C:\Windows\system32\svchost.exe
** 2296 628     svchost.exe     0xce0654d5a080  1       -       0       False   2024-09-10 05:30:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k utcsvc -p    C:\Windows\System32\svchost.exe
** 1316 628     svchost.exe     0xce0656dda080  7       -       0       False   2024-09-10 05:29:27.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalService -p -s EventSystem       C:\Windows\system32\svchost.exe
** 2340 628     svchost.exe     0xce065711c300  1       -       0       False   2024-09-10 05:30:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalService -p -s SstpSvc   C:\Windows\system32\svchost.exe
** 1576 628     svchost.exe     0xce0656ecf080  8       -       0       False   2024-09-10 05:29:32.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k NetworkService -p -s NlaSvc  C:\Windows\System32\svchost.exe
** 2552 628     wlms.exe        0xce06571dd080  2       -       0       False   2024-09-10 05:30:06.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\wlms\wlms.exe     C:\Windows\system32\wlms\wlms.exe       C:\Windows\system32\wlms\wlms.exe
** 1960 628     svchost.exe     0xce0657b79080  9       -       0       False   2024-09-10 05:41:50.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p -s PcaSvc    C:\Windows\system32\svchost.exe
** 1324 628     svchost.exe     0xce0656dd8080  2       -       0       False   2024-09-10 05:29:27.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s ProfSvc        C:\Windows\system32\svchost.exe
** 1584 628     svchost.exe     0xce0656ecd0c0  3       -       0       False   2024-09-10 05:29:32.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k netsvcs -p -s ShellHWDetection       C:\Windows\System32\svchost.exe
** 2868 628     svchost.exe     0xce06572d7080  14      -       0       False   2024-09-10 05:30:16.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k netsvcs      C:\Windows\System32\svchost.exe
** 3508 628     svchost.exe     0xce06570b2080  3       -       0       False   2024-09-10 05:32:06.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k appmodel -p -s StateRepository       C:\Windows\system32\svchost.exe
** 2360 628     svchost.exe     0xce06571872c0  4       -       0       False   2024-09-10 05:30:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p -s SysMain   C:\Windows\system32\svchost.exe
** 1468 628     svchost.exe     0xce0656e33080  3       -       0       False   2024-09-10 05:29:29.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s SENS   C:\Windows\system32\svchost.exe
** 956  628     svchost.exe     0xce0654f9b700  6       -       0       False   2024-09-10 05:29:15.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k DcomLaunch -p -s LSM C:\Windows\system32\svchost.exe
** 2492 628     svchost.exe     0xce0654e8b080  13      -       0       False   2024-09-10 05:30:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s WpnService     C:\Windows\system32\svchost.exe
** 4028 628     svchost.exe     0xce0656e30080  6       -       0       False   2024-09-10 05:32:36.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork -p -s DPS      C:\Windows\System32\svchost.exe
** 1728 628     svchost.exe     0xce0656eaa080  14      -       0       False   2024-09-10 05:29:35.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalServiceNoNetworkFirewall -p     C:\Windows\system32\svchost.exe
** 2244 628     msdtc.exe       0xce065784b080  9       -       0       False   2024-09-10 05:33:19.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\msdtc.exe C:\Windows\System32\msdtc.exe   C:\Windows\System32\msdtc.exe
** 1864 628     svchost.exe     0xce0656f87080  11      -       0       False   2024-09-10 05:29:39.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k LocalService -p -s netprofm  C:\Windows\System32\svchost.exe
** 2632 628     MsMpEng.exe     0xce06571e3080  7       -       0       False   2024-09-10 05:30:07.000000 UTC  N/A     \Device\HarddiskVolume1\ProgramData\Microsoft\Windows Defender\Platform\4.18.24070.5-0\MsMpEng.exe "C:\ProgramData\Microsoft\Windows Defender\platform\4.18.24070.5-0\MsMpEng.exe"    C:\ProgramData\Microsoft\Windows Defender\platform\4.18.24070.5-0\MsMpEng.exe
** 2760 628     svchost.exe     0xce0657293080  6       -       0       False   2024-09-10 05:30:12.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k smbsvcs -s LanmanServer      C:\Windows\System32\svchost.exe
** 1996 628     svchost.exe     0xce0654ec2080  3       -       0       False   2024-09-10 05:29:44.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted -p -s WinHttpAutoProxySvc      C:\Windows\system32\svchost.exe
** 2128 628     spoolsv.exe     0xce065700d5c0  8       -       0       False   2024-09-10 05:29:53.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\spoolsv.exe       C:\Windows\System32\spoolsv.exe C:\Windows\System32\spoolsv.exe
** 1236 628     svchost.exe     0xce0656de9080  2       -       0       False   2024-09-10 05:29:26.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalServiceNoNetwork -p     C:\Windows\system32\svchost.exe
** 2516 628     svchost.exe     0xce0654ecb080  5       -       0       False   2024-09-10 05:30:05.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k NetworkService -p -s WinRM   C:\Windows\System32\svchost.exe
** 1748 628     svchost.exe     0xce06571cf080  3       -       0       False   2024-09-10 05:31:54.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s TabletInputService        C:\Windows\System32\svchost.exe
*** 928 1748    ctfmon.exe      0xce0656f4f080  9       -       1       False   2024-09-10 05:31:57.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\ctfmon.exe        "ctfmon.exe"    C:\Windows\system32\ctfmon.exe
** 1628 628     svchost.exe     0xce0656ec1080  18      -       0       False   2024-09-10 05:29:32.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule       C:\Windows\system32\svchost.exe
*** 2472        1628    MicrosoftEdgeU  0xce06570b1080  4       -       0       True    2024-09-10 05:31:51.000000 UTC  N/A     \Device\HarddiskVolume1\Program Files (x86)\Microsoft\EdgeUpdate\MicrosoftEdgeUpdate.exe   "C:\Program Files (x86)\Microsoft\EdgeUpdate\MicrosoftEdgeUpdate.exe" /c  C:\Program Files (x86)\Microsoft\EdgeUpdate\MicrosoftEdgeUpdate.exe
*** 4648        1628    taskhostw.exe   0xce06574c8080  6       -       1       False   2024-09-10 06:14:58.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\taskhostw.exe     -       -
*** 1676        1628    taskhostw.exe   0xce0656e46080  6       -       1       False   2024-09-10 05:52:43.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\taskhostw.exe     taskhostw.exe   C:\Windows\system32\taskhostw.exe
*** 752 1628    taskhostw.exe   0xce06574c3080  4       -       1       False   2024-09-10 05:31:50.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\taskhostw.exe     taskhostw.exe {222A245B-E637-4AE9-A93F-A59CA119A75E}    C:\Windows\system32\taskhostw.exe
*** 4656        1628    wermgr.exe      0xce0657303080  1       -       0       False   2024-09-10 06:14:57.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\wermgr.exe        -       -
*** 3184        1628    taskhostw.exe   0xce0656ea7600  9       -       0       False   2024-09-10 06:15:00.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\taskhostw.exe     -       -
** 2784 628     svchost.exe     0xce0654ed3080  8       -       0       False   2024-09-10 05:30:13.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k NetSvcs -p -s iphlpsvc       C:\Windows\System32\svchost.exe
** 1124 628     svchost.exe     0xce0656d76080  2       -       0       False   2024-09-10 05:29:25.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted -p -s TimeBrokerSvc    C:\Windows\system32\svchost.exe
** 2276 628     svchost.exe     0xce06570a7080  9       -       0       False   2024-09-10 05:30:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k apphost -s AppHostSvc        C:\Windows\system32\svchost.exe
** 2404 628     svchost.exe     0xce065719a2c0  3       -       0       False   2024-09-10 05:30:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s TrkWks    C:\Windows\System32\svchost.exe
** 1892 628     svchost.exe     0xce0656ca4080  8       -       0       False   2024-09-10 05:33:30.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p -s UALSVC    C:\Windows\system32\svchost.exe
** 1512 628     svchost.exe     0xce0656dcc080  11      -       0       False   2024-09-10 05:29:31.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k NetworkService -p -s Dnscache        C:\Windows\system32\svchost.exe
** 1640 628     svchost.exe     0xce0656f9c080  4       -       1       False   2024-09-10 05:31:50.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k UnistackSvcGroup -s WpnUserService   C:\Windows\system32\svchost.exe
** 1132 628     svchost.exe     0xce0656d74080  9       -       0       False   2024-09-10 05:29:25.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted -p -s EventLog C:\Windows\System32\svchost.exe
** 2284 628     svchost.exe     0xce06570a5080  5       -       0       False   2024-09-10 05:30:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k NetworkService -p -s CryptSvc        C:\Windows\system32\svchost.exe
** 1520 628     svchost.exe     0xce0656e2c0c0  9       -       0       False   2024-09-10 05:29:31.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted -p     C:\Windows\system32\svchost.exe
** 1396 628     svchost.exe     0xce0656e44080  3       -       0       False   2024-09-10 05:29:29.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k LocalService -p -s nsi       C:\Windows\system32\svchost.exe
** 1144 628     svchost.exe     0xce0656d70080  3       -       0       False   2024-09-10 05:29:25.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted -p -s lmhosts  C:\Windows\System32\svchost.exe
** 3580 628     svchost.exe     0xce06571a1080  5       -       0       False   2024-09-10 05:32:02.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\svchost.exe       C:\Windows\system32\svchost.exe -k netsvcs -p -s TokenBroker    C:\Windows\system32\svchost.exe
* 820   508     fontdrvhost.ex  0xce0654f660c0  5       -       0       False   2024-09-10 05:29:12.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\fontdrvhost.exe   "fontdrvhost.exe"       C:\Windows\system32\fontdrvhost.exe
556     476     winlogon.exe    0xce0654eb60c0  4       -       1       False   2024-09-10 05:29:04.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\winlogon.exe      winlogon.exe    C:\Windows\system32\winlogon.exe
* 816   556     fontdrvhost.ex  0xce0654f67080  5       -       1       False   2024-09-10 05:29:12.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\fontdrvhost.exe   "fontdrvhost.exe"       C:\Windows\system32\fontdrvhost.exe
* 1016  556     dwm.exe 0xce0656c62080  16      -       1       False   2024-09-10 05:29:17.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\System32\dwm.exe   "dwm.exe"       C:\Windows\system32\dwm.exe
* 3908  556     userinit.exe    0xce06575f1080  0       -       1       False   2024-09-10 05:32:30.000000 UTC  2024-09-10 05:33:50.000000 UTC  \Device\HarddiskVolume1\Windows\System32\userinit.exe      -       -
** 3976 3908    explorer.exe    0xce0657486080  36      -       1       False   2024-09-10 05:32:32.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\explorer.exe       C:\Windows\Explorer.EXE C:\Windows\Explorer.EXE
*** 3628        3976    FTK Imager.exe  0xce065772d080  15      -       1       False   2024-09-10 06:09:42.000000 UTC  N/A     \Device\HarddiskVolume1\Program Files\AccessData\FTK Imager\FTK Imager.exe "C:\Program Files\AccessData\FTK Imager\FTK Imager.exe"         C:\Program Files\AccessData\FTK Imager\FTK Imager.exe
4200    4156    RegSvcs.exe     0xce06577c5080  4       -       1       True    2024-09-10 05:34:26.000000 UTC  N/A     \Device\HarddiskVolume1\Windows\Microsoft.NET\Framework\v4.0.30319\RegSvcs.exe     "C:\Users\admin\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\update.exe" C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegSvcs.exe
s18175  0xce06577c4080  314390  -       -       True    N/A     N/A     -       -       -

```
