# SOC L1 — Атаки: Детект на всех уровнях
# Формат: Вопрос | Ответ (разделитель |)
# Темы: Kerberos / NTLM / Lateral Movement / Network / Web / PrivEsc / Exfil / Email / Defense Evasion

---

## ══ KERBEROS АТАКИ ══

Kerberoasting — механика|Атакующий запрашивает TGS-тикеты для учёток с SPN (сервисных учёток). Тикеты зашифрованы паролем сервисной учётки → ломает оффлайн (hashcat -m 13100). Требует только доменного пользователя. Цель: получить plaintext пароль сервисной учётки, часто с высокими привилегиями.

Kerberoasting — детект на уровне хоста|Windows EID 4769 (TGS Request) + TicketEncryptionType = 0x17 (RC4-HMAC). В AES-среде любой RC4 TGS-запрос = красный флаг. Обращать внимание: много 4769 за короткое время от одной учётки к разным SPN.

Kerberoasting — детект на уровне сети|Сетевой анализ: Kerberos TGS-REQ пакеты от одного хоста к нескольким сервисным учёткам подряд. Wireshark: krb5 && kerberos.etype == 23 (RC4). NetFlow: всплеск Kerberos трафика (порт 88) с хоста без DA прав.

Kerberoasting — детект в SIEM|Правило: EID 4769, TicketEncryptionType=0x17, ServiceName не содержит "$" и не krbtgt. Threshold: >5 RC4 TGS запросов от одной учётки за 60 секунд = сканирование. Обогатить: к каким SPN обращался? Вычислить baseline TGS по учётке.

Kerberoasting — защита и реакция L1|1. Подтвердить: RC4 TGS в AES-среде. 2. Сколько SPN запрошено? 3. С какого хоста — проверить хост на IoC. 4. Эскалировать L2: аудит паролей сервисных учёток, Managed Service Accounts, AES-only для krbtgt. L2 меняет пароли всех SPN учёток.

AS-REP Roasting — механика и отличие от Kerberoasting|Атакует учётки с отключённой Kerberos pre-authentication (DONT_REQ_PREAUTH). Атакующий запрашивает AS-REP без предварительной аутентификации → получает зашифрованный blob → ломает оффлайн (hashcat -m 18200). Не требует аутентификации в домене вообще — анонимно!

AS-REP Roasting — детект (EID 4768)|EID 4768: AS-REQ без pre-auth → PreAuthType = 0. Подозрительно: запросы к учёткам с DONT_REQ_PREAUTH от нестандартного хоста. SIEM правило: EID 4768, PreAuthType=0, множественные запросы с одного IP. Также: аудит учёток с этим флагом через PowerShell: Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true}.

Pass-the-Ticket (PtT) — механика|Атакующий извлекает Kerberos-тикет (TGT или TGS) из памяти (mimikatz kerberos::list/export) и инжектирует в свою сессию (kerberos::ptt). Тикет действителен без знания пароля пока не истечёт (TGT = 10 часов по умолчанию).

Pass-the-Ticket — детект хост уровень|Sysmon EID 10: доступ к lsass.exe для извлечения тикетов. EID 4624 LogonType=9 (RunAs с другими кредами) + NTLM аутентификация после инъекции. Аномалия: та же учётка аутентифицируется с разных хостов одновременно. Mimikatz: EID 1 (запуск) + EID 10 (lsass).

Pass-the-Ticket — детект сеть и SIEM|Kerberos TGS-REQ без предшествующего TGT обновления с того же хоста. Аномалия временных меток тикета: тикет создан раньше чем пользователь залогинился. SIEM: один TGT используется с разных IP (перемещение тикета). Корреляция EID 4624 по LogonID → нетипичный хост.

Golden Ticket — механика и что нужно|Поддельный TGT, подписанный NTLM-хешем krbtgt. Требует: SID домена, имя домена, NTLM хеш krbtgt (из DCSync или NTDS.dit дампа). Действителен 10 лет по умолчанию. Mimikatz: kerberos::golden /user:любой /domain:/ /sid: /krbtgt:<hash>. Позволяет аутентифицироваться как любой пользователь домена.

Golden Ticket — детект|EID 4768/4769 с аномалиями: несуществующий пользователь в домене, нестандартный срок тикета (>7 дней), RC4 шифрование в AES-среде. EID 4672 (спец-привилегии) без предшествующего EID 4768. Аутентификация на DC без AS-REQ к тому же DC. CRITICAL: почти нет надёжного детекта без специальных honeytokens.

Golden Ticket — реакция|Единственное эффективное средство: смена пароля krbtgt ДВАЖДЫ (первый раз инвалидирует текущие TGT, второй — удаляет все golden-тикеты на основе старого хеша). Между сменами: 10+ часов (TTL TGT). DFIR: найти как получили krbtgt (DCSync? ntds.dit dump?).

Silver Ticket — механика и отличие от Golden|Поддельный TGS для конкретного сервиса. Нужен только NTLM хеш машинной учётки (не krbtgt). Не логируется на DC! Прямое обращение к сервису (CIFS, HOST, HTTP). Менее мощный чем Golden, но стелснее. Пример: Silver Ticket для CIFS → доступ к файловым ресурсам машины.

Silver Ticket — детект|Практически нет логов на DC (нет AS-REQ, нет TGS-REQ на DC). Детект на целевом сервисе: EID 4624 с аномальными полями (нестандартный user, нет соответствующего 4768/4769 на DC). Sysmon на машине-жертве: соединение к целевому сервису без предшествующей Kerberos аутентификации. Honey tokens на чувствительных ресурсах.

Overpass-the-Hash (Pass-the-Key) — механика|Использование NTLM хеша для получения Kerberos TGT (а не для прямой NTLM аутентификации). Mimikatz: sekurlsa::pth /user:admin /domain: /ntlm:<hash>. Результат: полноценная Kerberos сессия. Отличие от PtH: использует Kerberos, не NTLM → обходит NTLM-блокировки.

Overpass-the-Hash — детект|EID 4768 (TGT запрос) где тип шифрования RC4 (0x17) — подозрительно в AES-среде. EID 4624 LogonType=9. Sysmon EID 10: доступ к lsass для извлечения NTLM хеша перед выпуском TGT. Anomaly: RC4 AS-REQ от хоста/учётки которые обычно используют AES.

---

## ══ NTLM АТАКИ ══

Pass-the-Hash (PtH) — механика|Использование NTLM хеша вместо пароля для сетевой аутентификации. Mimikatz: sekurlsa::pth, CrackMapExec: -H <hash>. Работает только для NTLM (не Kerberos). Хеш берётся из lsass (sekurlsa::logonpasswords), SAM (lsadump::sam) или NTDS.dit.

Pass-the-Hash — детект хост уровень|EID 4624 LogonType=3 (сетевой) + AuthenticationPackageName=NTLM + TargetUserName привилегированной учётки. Аномалия: нет предшествующего интерактивного входа (LogonType=2/10) для той же учётки. Sysmon EID 10: доступ к lsass.exe → извлечение хеша.

Pass-the-Hash — детект сеть|SMB трафик (порт 445) с NTLM аутентификацией от хоста где данная привилегированная учётка никогда не работала. NetFlow: всплеск SMB соединений с одного хоста ко многим (горизонтальное движение). Wireshark: ntlmssp.auth в SMB пакетах.

Pass-the-Hash — SIEM правило|Корреляция: EID 4624 (LogonType=3, NTLM) + без EID 4648 (explicit credentials) с этого хоста + без EID 4634 (logout) предшествующей сессии + TargetUserName = DA/Admin. Порог: один хост аутентифицируется как один DA к N хостам за 5 минут.

NTLM Relay — механика (Responder)|Responder перехватывает NTLM authentication запросы (LLMNR, NBT-NS, MDNS broadcasts). Жертва пытается найти несуществующий хост → broadcast → Responder отвечает "я здесь" → жертва отправляет NTLM challenge/response → ntlmrelayx.py пересылает на целевой сервис. Не нужно ломать хеш!

NTLM Relay — детект L2 сеть|LLMNR/NBT-NS broadcast запросы на UDP 5355/137 → ответы не от DNS. Аномалия: устройство отвечает на LLMNR запросы которые не адресованы ему. Детект: IDS правило на несанкционированные LLMNR ответы. Защита: отключить LLMNR/NBT-NS через GPO.

NTLM Relay — детект L7 хост|EID 4624 LogonType=3 NTLM от нетипичного хоста. Событие: компьютер X аутентифицировался на сервисе Y с использованием кредов пользователя Z — но Z не за компьютером X. Sysmon EID 3: ntlmrelayx или impacket соединение к DC/сервису. Артефакт: secretsdump, добавление пользователя через relay.

Захват NTLMv2 хешей (Responder capture)|Responder.py перехватывает NTLMv2 хеши в NTLM challenge → пробросить в hashcat (-m 5600). Хеши не используются напрямую как PtH (NTLMv2), но поддаются dictionary attack. Детект: аномальные LLMNR/NBT-NS ответы, много 4776 с ошибками, Wireshark: broadcast ответы с нестандартного MAC.

---

## ══ LATERAL MOVEMENT (ДЕТАЛЬНО) ══

PsExec — механика и артефакты|PsExec копирует exe на ADMIN$ share (\\target\ADMIN$\PSEXESVC.EXE), устанавливает сервис PSEXESVC, запускает его → интерактивный shell. Требует: Admin права, SMB 445, доступ к ADMIN$. Оставляет: EID 7045 (PSEXESVC сервис), EID 5140 (ADMIN$ share access), exe на диске.

PsExec — детект хост (EventLog + Sysmon)|EID 7045: ServiceName = PSEXESVC, ImagePath = %SystemRoot%\PSEXESVC.EXE. EID 5140: ShareName = \*\ADMIN$. EID 4624 LogonType=3 (предшествует). Sysmon EID 1: PSEXESVC.EXE → cmd.exe (дочерний shell). EID 4688: CommandLine показывает команды выполненные через PsExec.

PsExec — детект сеть|SMB порт 445: передача файла (PSEXESVC.EXE), затем RPC (135) для создания сервиса. NetFlow: два соединения Source→Target: сначала 445 (копирование), потом 445 снова (командный канал). Wireshark: SMB2 Create Request к ADMIN$ с именем PSEXESVC.

PsExec — SIEM корреляция|EID 5140 (ADMIN$ access) + EID 7045 (PSEXESVC) + EID 4624 LogonType=3 от того же SourceIP в течение 60 секунд → высоковероятный PsExec. Дополнительно: EID 4634 (disconnect) после завершения работы. PSEXESVC в нерабочее время = высокий приоритет.

WMI Lateral Movement — механика|wmic /node:TARGET process call create "CMD". Использует DCOM (порт 135 → динамический). Родитель на удалённой машине: WmiPrvSE.exe → дочерний процесс (cmd.exe, powershell.exe). Не копирует файлы — только выполняет команды. Менее заметен чем PsExec (нет сервиса).

WMI Lateral Movement — детект хост|EID 4688/Sysmon EID 1: WmiPrvSE.exe (родитель) → cmd.exe/powershell.exe (ребёнок) — ключевой признак. EID 4624 LogonType=3 NTLM/Kerberos предшествует. EID 4648: явное использование кредов. Sysmon EID 3: WmiPrvSE.exe устанавливает соединение (если payload делает сетевые вызовы).

WMI Lateral Movement — детект сеть|DCOM: соединение на порт 135 (RPC Endpoint Mapper), затем динамический порт 49152+ для WMI. NetFlow: RPC трафик Source→Target. Wireshark: DCOM/MSRPC IWbemServices::ExecMethod. Детект в firewall: блокировка 135 между рабочими станциями (lateral movement prevention).

WinRM Lateral Movement — механика|PowerShell Remoting: Invoke-Command -ComputerName TARGET -ScriptBlock {cmd}. evil-winrm -i TARGET -u user -p pass. HTTP порт 5985 (или HTTPS 5986). Логируется на целевой машине как EID 4624 LogonType=3.

WinRM Lateral Movement — детект|EID 4624 LogonType=3 + порт 5985/5986 (из firewall логов). Sysmon EID 3: wsmprovhost.exe устанавливает соединение. EID 4688: wsmprovhost.exe как родитель дочернего процесса. PowerShell EID 4104 (Script Block): команды выполненные через WinRM. Аномалия: WinRM трафик между рабочими станциями (не к серверам).

RDP Lateral Movement — механика и атрибуты|mstsc.exe → целевой хост порт 3389. LogonType=10 на целевой машине. Credentials кешируются в lsass (без NLA). Техники: RDP Pass-the-Hash (restricted admin mode), SharpRDP, Mimikatz RDP hijacking (ts::multirdp). Артефакты: EID 4624 LogonType=10, EID 1149 (RDP auth success), ShimCache/Prefetch.

RDP Lateral Movement — детект хост и сеть|EID 4624 LogonType=10 + SourceIP нетипичный. EID 1149 (Microsoft-Windows-TerminalServices-RemoteConnectionManager: User authentication succeeded). Sysmon EID 3: mstsc.exe → порт 3389 к нетипичному хосту. NetFlow: TCP 3389 между внутренними хостами в нерабочее время. SIEM: chain RDP hops (хост A → хост B → хост C).

SMB Lateral Movement (net use, copy) — детект|EID 5140 (шара IPC$, ADMIN$, C$). EID 4624 LogonType=3. NetFlow: SMB порт 445 горизонтально между хостами. Sysmon EID 11: файлы скопированы в \Windows\Temp\, \Users\Public\ на удалённой машине. Признак: доступ к ADMIN$ (административная шара) = всегда обоснованный алерт.

DCOM Lateral Movement — механика|Использует DCOM объекты для удалённого выполнения кода. Примеры: ShellWindows (ShellExecute), Excel.Application (DDEInitiate), MMC20.Application (ExecuteShellCommand). Труднее детектировать чем WMI. Родитель на целевой: explorer.exe или mmc.exe → cmd/powershell.

DCOM Lateral Movement — детект|EID 4688/Sysmon EID 1: explorer.exe → cmd.exe с внешней CommandLine (аномалия!). Sysmon EID 3: DCOM порты 135 + динамические. EID 4624 LogonType=3. Родительский процесс не является типичным для дочернего: mmc.exe → powershell.exe = подозрительно.

---

## ══ AD АТАКИ (РАСШИРЕННЫЕ) ══

DCSync — механика детально|Mimikatz lsadump::dcsync использует MS-DRSR (Directory Replication Service Remote Protocol) для запроса репликации объектов AD, включая NTLM хеши всех пользователей (как если бы был второй DC). Требует прав: DS-Replication-Get-Changes + DS-Replication-Get-Changes-All.

DCSync — детект хост|EID 4662 (Access to AD Object) на DC: SubjectUserName = атакующий, ObjectType = {19195a5b...} (DS-Replication-Get-Changes-All). Критично: запрос репликации с хоста который НЕ является DC. EID 4624 предшествует (как DA или с delegated правами). Mimikatz в Sysmon EID 1.

DCSync — детект сеть|MS-DRSR трафик: MSRPC интерфейс {e3514235-4b06-11d1-ab04-00c04fc2dcd2}. Wireshark фильтр: drsuapi. Порт 135 (endpoint mapper) + динамический. Детект: хост не-DC отправляет DRSGetNCChanges запрос к DC. NDR/NTA решения: детект DRSUAPI от не-DC хостов.

LDAP Enumeration / BloodHound — детект|Массовые LDAP запросы к DC: EID 1644 (LDAP query с высокой стоимостью). Sysmon EID 3: SharpHound.exe или powershell → порт 389/636 к DC. Аномалия: рядовой пользователь делает сотни LDAP запросов (GetAllGroups, GetAllUsers, GetAllComputers) за короткое время.

AD Enumeration через встроенные команды — детект|net group "Domain Admins" /domain, Get-ADUser, nltest /dclist. Sysmon EID 1: net.exe, net1.exe с аргументами domain. EID 4688: те же аргументы. Родительский процесс: cmd.exe/powershell.exe запущен от w3wp.exe (webshell) или revshell.exe = critical. Порог алерта: N AD enum команд за M минут.

Token Impersonation — механика|Дублирование токена другого процесса (SeImpersonatePrivilege или SeAssignPrimaryTokenPrivilege). Инструменты: Incognito, juicy-potato, rogue-potato, printspoofer. Используется для PE: SYSTEM → Domain Admin, или IIS (Network Service) → SYSTEM. Часто встречается в среде WebShell.

Token Impersonation — детект|EID 4624 LogonType=3/9 с ImpersonationLevel=Impersonation. Sysmon EID 8 (CreateRemoteThread) или EID 10 (Process Access) между процессами. EID 4672 (Special Logon) после стандартного пользователя = PE. Powershell: Get-Process → проверить токены процессов через ProcessExplorer. Аномалия: IIS рабочий процесс w3wp.exe получает привилегии SYSTEM.

---

## ══ PRIVILEGE ESCALATION WINDOWS ══

UAC Bypass — механика|User Account Control можно обойти через: fodhelper.exe (реестр HKCU\...\fodhelper\shell\open\command), eventvwr.mms (HKCU hijack), DiskCleanup (COM elevation), sdclt.exe. Результат: процесс запускается с высокими привилегиями без UAC промпта.

UAC Bypass — детект|Sysmon EID 12/13: запись в HKCU\Software\Classes\...\shell\open\command. EID 4688/Sysmon EID 1: fodhelper.exe → дочерний процесс с High integrity level. ProcessIntegrityLevel в Sysmon: неожиданно High для пользовательского процесса. SIEM: цепочка reg write → запуск auto-elevated процесса → дочерний процесс.

AlwaysInstallElevated — механика и детект|Если GPO: AlwaysInstallElevated=1 в HKLM и HKCU → любой MSI файл запускается с SYSTEM правами. Создать malware.msi → запустить. Детект: EID 1040/1042 (Windows Installer) с SYSTEM уровнем, Sysmon EID 1 (msiexec.exe → cmd/powershell SYSTEM), проверить GPO: Get-ItemProperty HKCU:\Software\Policies\Microsoft\Windows\Installer.

DLL Search Order Hijacking — механика|Приложение ищет DLL в порядке: 1)KnownDLLs, 2)директория exe, 3)System32, 4)Windows, 5)PATH. Атакующий помещает вредоносную DLL в директорию с записью раньше легитимной. Пример: Teams.exe ищет VERSION.dll → положить вредоносную в ту же папку.

DLL Search Order Hijacking — детект|Sysmon EID 7 (ImageLoad): загрузка unsigned DLL из нестандартного места (не System32). Process Monitor: DLL NOT FOUND → DLL LOADED из неожиданного места. EID 4688: процесс запустился и сразу загрузил подозрительную DLL. Autoruns: DLL hijack entries. Инструмент: PowerSploit Find-PathDLLHijack.

Potato Attacks (JuicyPotato, PrintSpoofer, RoguePotato) — концепция|Используют SeImpersonatePrivilege (есть у IIS, SQL Server, Network Service) для PE до SYSTEM. PrintSpoofer: вызывает Spooler сервис для подключения к named pipe атакующего → impersonate. JuicyPotato: DCOM активация CLSID с высокими правами. Частый вектор: WebShell → IIS token → PrintSpoofer → SYSTEM.

Potato Attacks — детект|Sysmon EID 17/18 (Named Pipe): создание нестандартного named pipe, соединение Spooler сервиса к нему. EID 4624 LogonType=3/9: impersonation. Sysmon EID 1: PrintSpoofer.exe, JuicyPotato.exe (или обфусцированный). Аномалия: w3wp.exe/mssql.exe → SystemProcess PE через named pipe.

---

## ══ PRIVILEGE ESCALATION LINUX ══

Linux PrivEsc: sudo -l — разведка и детект|sudo -l показывает какие команды пользователь может выполнять через sudo без пароля. Классика: sudo vim/nano/find/python3/bash → shell с правами root. Детект: auditd на sudo -l (execve sudo), /var/log/auth.log: "COMMAND" записи sudo. Аномалия: sudo для интерпретаторов (vim, python, perl) = немедленная PE.

Linux PrivEsc: Cron Jobs — механика|Просмотр /etc/crontab и /etc/cron.d/ для поиска задач запущенных от root с записываемыми файлами. Атака: перезаписать скрипт вызываемый root-cron → PE. Если cron вызывает PATH-зависимый бинарь → подмена в PATH. Детект: auditd на запись в /etc/cron*, inotifywait на изменения скриптов вызываемых cron.

Linux PrivEsc: SUID бинари — детект расширенный|find / -perm -4000 -type f 2>/dev/null. Опасные SUID: /bin/bash -p, find, vim, python, perl, cp. Детект: auditd правило: -a always,exit -F arch=b64 -S chmod -F perm=4000 (изменение SUID). Baseline: список SUID при установке системы vs текущий → diff. Sysmon Linux: execve /bin/bash -p.

Linux PrivEsc: Writable /etc/passwd — механика|Если /etc/passwd имеет права на запись → добавить строку: hacker:$(openssl passwd -1 password):0:0:root:/root:/bin/bash. Вход как hacker = root (UID=0). Детект: auditd на запись в /etc/passwd (-w /etc/passwd -p wa), системы FIM (File Integrity Monitoring), tripwire/AIDE.

Linux PrivEsc: PATH Hijacking|Скрипт от root вызывает команду без абсолютного пути (например: cp вместо /bin/cp). Атакующий создаёт /tmp/cp с вредоносным кодом и добавляет /tmp в начало PATH. При запуске root-скрипта выполняется /tmp/cp. Детект: auditd на execve в /tmp, аномальный PATH в sudo env, мониторинг относительных вызовов в cron скриптах.

Linux PrivEsc: LXD/Docker Escape|Пользователь в группе lxd/docker → создать привилегированный контейнер с монтированием / хоста → chroot → root на хосте. Детект: auditd на docker run --privileged, проверка /var/log/docker.log, мониторинг Docker socket /var/run/docker.sock, аномальный контейнер с bind mount /.

---

## ══ СЕТЕВЫЕ АТАКИ (ДЕТАЛЬНО ПО УРОВНЯМ) ══

ARP Spoofing/Poisoning — механика L2|Атакующий рассылает ложные ARP Reply: "IP шлюза имеет мой MAC". ARP таблицы жертв обновляются. Трафик жертв идёт через атакующего (MITM). Инструменты: arpspoof, ettercap, Scapy. Работает только внутри L2 сегмента (broadcast domain).

ARP Spoofing — детект L2 (сеть)|Анализ ARP трафика: Wireshark arp.duplicate-address-detected. Признаки: один MAC претендует на несколько IP, незапрошенные ARP Reply (Gratuitous ARP), высокая частота ARP Reply. NetFlow: резкое изменение MAC для IP шлюза. Защита: Dynamic ARP Inspection (DAI) на коммутаторах, статические ARP записи для шлюза.

ARP Spoofing — детект в SIEM/хост|SIEM: corr. IP→MAC mapping change в DHCP логах. Endpoint: изменение ARP кеша (arp -a до и после). Признак MITM: latency увеличилась, SSL cert ошибки. IDS Suricata: alert arp (op 2) → anomaly rule. Security коммутатора: IPSG (IP Source Guard) алерты.

DNS Spoofing / Cache Poisoning — механика L3|Атакующий отправляет поддельный DNS ответ раньше легитимного сервера (race condition) или отравляет кеш рекурсивного резолвера (Kaminsky атака: угадать transaction ID + source port). Жертва получает поддельный A-record → перенаправляется на сервер атакующего.

DNS Cache Poisoning — детект L3/L7|DNS логи: множественные ответы для одного запроса (первый пришёл легитимный, второй — поддельный). Аномальный TTL (очень низкий = быстрая смена записей). DNS ответ от нелегитимного IP (не настроенный NS). Защита: DNSSEC (валидация подписи), рандомизация source port, 0x20 encoding. Детект: RPZ (Response Policy Zones) в bind/unbound.

SYN Flood — механика L4|Атакующий отправляет много TCP SYN пакетов со spoofed IP. Сервер отправляет SYN-ACK и ждёт ACK (half-open connection). Таблица half-open connections переполняется → сервер перестаёт принимать новые соединения (DoS). Защита: SYN cookies (сервер не хранит состояние до ACK).

SYN Flood — детект L4|NetFlow: аномальный объём SYN пакетов к одному порту. Firewall статистика: много TCP SYN без ACK. Wireshark: tcp.flags.syn==1 && tcp.flags.ack==0 — тысячи в секунду. Счётчик ss -s (Linux): LISTEN overflow, много SYN-RECV. SIEM: threshold SYN пакетов/сек от одного IP → firewall ACL.

DNS Amplification DDoS — механика L3/L4|Атакующий отправляет DNS ANY запросы со spoofed IP жертвы. DNS сервер отвечает большим ответом (amplification 50-70x) на IP жертвы. Много DNS серверов = DDoS на жертву. Используют open DNS resolvers. Защита: Response Rate Limiting (RRL) на DNS серверах, BCP38 (anti-spoofing).

DNS Amplification — детект|Firewall/NetFlow: большой входящий UDP 53 от множества IP (open resolvers) к одному IP (жертва). DNS логи сервера: аномальное количество ANY или TXT запросов с одного IP (если сервер используется как усилитель). SIEM: всплеск UDP 53 трафика. Индикатор: IP источник нигде раньше не встречался + ответы непропорционально большие.

VLAN Hopping — механика L2|Атака 1: Switch Spoofing — порт атакующего становится trunk портом, получает трафик всех VLAN. Атака 2: Double Tagging — добавление двух 802.1Q тегов, коммутатор снимает первый (native VLAN) и перенаправляет во второй VLAN. Защита: выключить DTP на access портах, изменить native VLAN.

VLAN Hopping — детект|CDP/LLDP логи: новый trunk neighbor появился на access порте. DTP negotiation: атакующий отправляет DTP пакеты для включения trunk. Netflow: трафик с хоста доступный только из другого VLAN. Syslog коммутатора: native VLAN mismatch, trunk creation on access port. Защита: switchport nonegotiate + explicit VLAN assignment.

---

## ══ EMAIL АТАКИ (ДЕТАЛЬНО) ══

Business Email Compromise (BEC) — типы|1. CEO Fraud: подделка директора → приказ перевести деньги. 2. Invoice Scam: поддельный счёт от поставщика. 3. Account Takeover: взломанный почтовый ящик для рассылки. 4. Lawyer Impersonation: юрист требует конфиденциального перевода. Детект: аномальные правила пересылки, внешние пересылки.

BEC — детект хост/почта|Почтовые логи: создание правил пересылки на внешние адреса (ExchangeAdmin или PowerShell). O365 Unified Audit Log: New-InboxRule с ForwardTo на внешний домен. Аномалия: почтовый ящик CEO/CFO получает доступ с нового IP/геолокации. Внезапные правила удаления писем (скрытие следов).

Email Spoofing без домена — защита сводная|SPF: проверяет IP сервера-отправителя (защита MAil FROM). DKIM: проверяет цифровую подпись заголовков (защита контента и From). DMARC: выравнивание SPF/DKIM с заголовком From + политика (none/quarantine/reject). Все три нужны! SPF без DMARC = легко обойти через другой домен.

HTML Smuggling — механика|Вредоносный payload (EXE/ZIP) кодируется в Base64 и встраивается в HTML файл. JavaScript при открытии файла декодирует и автоматически сохраняет на диске (Blob API: URL.createObjectURL). Обходит Email-шлюзы: вложение HTML безопасно, payload создаётся только в браузере. Детект: EID 4688: браузер → explorer → запуск скачанного файла.

HTML Smuggling — детект|Sysmon EID 11: создание exe/zip файла процессом браузера (chrome.exe, firefox.exe) через downloads. EID 1: браузер → explorer.exe → rundll32/wscript/cmd (необычная цепочка). Анализ HTML вложения: большие Base64 блоки в script теге + Blob/ArrayBuffer API. ANY.RUN: открыть HTML файл → проверить created files.

Macro-enabled Documents — детект детально|EID 4688/Sysmon EID 1: winword.exe или excel.exe → cmd.exe/powershell.exe/wscript.exe. Sysmon EID 3: офис процесс делает сетевое соединение (download payload). EID 4104: PowerShell ScriptBlock — декодированное содержимое. AMSI: попытка обойти. Email шлюз: блокировать .xlsm, .docm, .doc с макросами. Sandbox: ANY.RUN на документ.

---

## ══ WEB АТАКИ ══

SQL Injection — механика по типам|Classic: ' OR '1'='1 → auth bypass. Error-based: извлечение данных через сообщения об ошибках. Blind (boolean): TRUE/FALSE запросы. Blind (time): AND SLEEP(5). UNION-based: UNION SELECT username,password FROM users. OOB: xp_cmdshell, DNS exfil (load_file → DNS запрос).

SQL Injection — детект L7|WAF логи: символы ' " -- ; OR AND в параметрах. Паттерны: UNION SELECT, SLEEP(), INFORMATION_SCHEMA, xp_cmdshell в URL/Body. Web server логи (IIS/Apache/nginx): 500 ошибки при проверке, аномальные запросы с SQL синтаксисом. SIEM: threshold SQL ошибок с одного IP. sqlmap User-Agent в логах.

SQL Injection — детект хост|MSSQL: EID 18456 (ошибки auth), ошибки синтаксиса в SQL Server logs. xp_cmdshell включён: EID 4688 (sqlservr.exe → cmd.exe). Linux/MySQL: /var/log/mysql/error.log. Sysmon EID 1: mysqld.exe/sqlservr.exe → cmd.exe = RCE через SQL. OOB: DNS запросы от DB сервера = exfil.

XSS (Cross-Site Scripting) — типы и детект|Reflected: payload в URL/params, выполняется сразу. Stored: хранится в DB, выполняется у всех. DOM-based: через JS DOM manipulation. Детект L7: WAF на <script>, javascript:, onerror=, onload=. Application логи: необычные символы в параметрах. CSP header нарушения (browser console). Burp Suite scan → активный сканер.

SSRF (Server-Side Request Forgery) — механика|Приложение делает запрос к URL указанному пользователем. Атакующий указывает: http://169.254.169.254/ (AWS metadata), http://localhost:6379/ (Redis), http://internal-db:5432/. Результат: доступ к внутренней инфраструктуре через сервер приложения.

SSRF — детект|Веб-логи: запросы к /latest/meta-data/, localhost, 127.0.0.1, 169.254.x.x, 0.0.0.0, 10.x.x.x в user-controlled параметрах (url=, redirect=, img=). Firewall: исходящие соединения от веб-сервера к внутренним IP (аномалия). Sysmon EID 3: w3wp.exe/python/java → внутренний адрес. IMDSv2 логи в AWS: запросы к metadata service.

LFI/Path Traversal — механика и детект|../../../etc/passwd, ....//....//etc/passwd, %2e%2e%2f. Чтение произвольных файлов. LFI + php wrapper: php://filter/read=convert.base64-encode/resource=/etc/passwd. Детект: /../ или %2e%2e в URL (WAF правило). Web логи: 200 ответ на запрос с ../ = успешный LFI. Аномальный path в параметре = триаж.

WebShell — загрузка и детект L7|POST запрос для загрузки (200 OK) → GET/POST запрос к загруженному файлу (200 OK + cmd output). Логи IIS/Apache: POST к *.aspx/*.php в директории uploads, затем GET с cmd= параметром. Признак: один и тот же IP загружает файл и сразу обращается к нему.

WebShell — детект хост|Sysmon EID 1: w3wp.exe (IIS) / httpd (Apache) → cmd.exe / powershell.exe / bash — ГЛАВНЫЙ ПРИЗНАК. EID 4688: те же. EID 11: создание файлов в webroot. Антивирус: сигнатура China Chopper, b374k, r57. FIM: изменение файлов в /var/www/ или C:\inetpub\wwwroot\.

---

## ══ DATA EXFILTRATION (ДЕТАЛЬНО) ══

DNS Tunneling — многоуровневый детект|L7/DNS: impacket-sniffer, dnscap — видят запросы. L3/NetFlow: объём DNS трафика с хоста >> baseline. L7/SIEM: Zeek dns.log — поле query length > 60 chars, тип TXT/NULL. Хост: Sysmon EID 22 (DNS Query) — длинные поддомены, высокая частота к одному домену. Инструменты детекта: Zscaler DNS security, RITA beaconing.

HTTP Exfiltration — детект всех уровней|L7/прокси-логи: большие POST к нетипичным внешним URL, нестандартные User-Agent. L4/NetFlow: аномальный исходящий объём с хоста с данными. L3/Firewall: исходящие HTTP/HTTPS к IP ранее не встречавшимся. Хост: Sysmon EID 3 (network connect) от необычного процесса + EID 1 (создан zip/rar/архив прямо перед). Корреляция: File Create (EID 11) → Network Connect (EID 3) одним процессом.

ICMP Tunneling — детект|L3/NetFlow: ICMP трафик с хоста >> baseline (норма: 0-10 pings). Wireshark: ICMP payload больше стандартного ping (8 байт data), высокая энтропия в payload, нестандартные ICMP types (не 8/0). Firewall: исходящий ICMP на внешний IP которого нет в whitelist. Хост: Sysmon EID 3 (тип протокола ICMP = 1).

Rclone Exfiltration — детект|Rclone — легитимный инструмент синхронизации который атакующие используют для exfil в S3/GDrive/Mega. Сигналы: Sysmon EID 1 (rclone.exe с параметрами remote:) + EID 3 (большой исходящий трафик к облачным провайдерам). Firewall: большой upload к storage.googleapis.com, s3.amazonaws.com. CLI: rclone config может хранить creds в %APPDATA%\rclone.

---

## ══ DEFENSE EVASION ══

AMSI Bypass — механика|AMSI (AntiMalware Scan Interface) перехватывает PowerShell/VBScript до выполнения. Обходы: Patch amsiInitFailed в памяти ([Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)). Рефлексия, String concatenation, обфускация.

AMSI Bypass — детект|EID 4104 (Script Block Logging): несмотря на bypass, Windows иногда логирует. ETW (Event Tracing): Microsoft-Antimalware-Engine провайдер. Поведенческий: процесс загружает amsi.dll → WriteProcessMemory к amsi.dll (patch). Sysmon EID 7: amsi.dll загружена подозрительным процессом. SIEM: powershell + AmsiUtils + amsiInitFailed в одном блоке.

ETW Patching — механика|Event Tracing for Windows — основа большинства логов Windows. Атакующий патчит EtwEventWrite() в ntdll.dll → return 0 без записи событий. Результат: Sysmon/Windows Events перестают работать для этого процесса. Скрытная атака — сложно детектировать.

ETW Patching — детект|Sysmon EID 7: запись в ntdll.dll (ImageLoad + WriteProcessMemory). Baseline: количество Sysmon событий от процесса резко падает (gap в временной линии). Kernel Driver: ETW провайдер уровня ядра труднее запатчить. Velociraptor: hunt на модификации ntdll/EtwEventWrite. Аномалия: процесс запустился → нет последующих событий Sysmon.

Log Tampering — детект|EID 1102: очистка Security журнала. EID 104 (System): очистка System журнала. Резкий gap во временной шкале событий. Аномалия: последний EID 4688 в 22:05, следующий — 06:30 (атакующий работал 8 часов без следов). wevtutil cl Security = требует SeSecurityPrivilege. SIEM: forward логи в реальном времени → нельзя удалить ретроспективно.

Timestomping — детект через NTFS|$STANDARD_INFORMATION (SI) изменяемый → атакующий меняет. $FILE_NAME (FN) обновляется только при rename/move → сложнее изменить. Расхождение SI vs FN временных меток = timestomping. Инструменты: Autopsy, MFT Analyzer, log2timeline. Парадокс: файл создан в 2019, но в MFT запись создана в 2025 → SI подделан.

Process Injection (общая схема) — типы|DLL Injection: OpenProcess + VirtualAllocEx + WriteProcessMemory + CreateRemoteThread(LoadLibraryA). Process Hollowing: создать svchost SUSPENDED → NtUnmapViewOfSection → WriteProcessMemory payload → ResumeThread. Thread Hijacking: OpenThread + SuspendThread → modify EIP → ResumeThread. Reflective DLL: DLL сама загружает себя в память без LoadLibraryA.

Process Injection — детект сводный|Sysmon EID 8 (CreateRemoteThread): SourceImage → TargetImage. EID 10 (ProcessAccess): PROCESS_VM_WRITE + PROCESS_VM_READ + PROCESS_CREATE_THREAD. EID 7 (ImageLoad): unsigned DLL загружена в легитимный процесс. Аномалия: svchost.exe без родителя services.exe + высокая энтропия PE секций = process hollowing. Volatility malfind: RWX регионы в процессах.

Obfuscated PowerShell — детект сводный|EID 4104 (Script Block Logging ВКЛЮЧИТЬ!): логирует ДЕКОДИРОВАННЫЙ код. AMSI: проверяет до выполнения. Признаки: [char] конкатенация, iex("".join(...)), -EncodedCommand, gzip+base64. Инструмент: Revoke-Obfuscation (PowerShell анализ). Behavioural: powershell → network connection → file drop = IOA цепочка.

---

## ══ RANSOMWARE — ПОЛНАЯ ЦЕПОЧКА ДЕТЕКТА ══

Ransomware: Stage 1 — Initial Access|Фишинг с вложением (macros) или эксплойт публичного сервиса. Детект: Email шлюз на .docm/.xlsm, Sysmon EID 1: winword.exe → powershell.exe, EID 3: офисное приложение → сетевое соединение (download dropper). Ключ: поймать здесь = нет ущерба.

Ransomware: Stage 2 — Persistence & Discovery|После initial access: registry Run key (EID 13), Scheduled Task (EID 4698). Разведка: net view /all, ipconfig /all, whoami /groups. Детект: Sysmon EID 13 (Run key), EID 4688 (discovery команды), SharpHound (LDAP запросы). Сигнал: офисный процесс → сетевые запросы → discovery команды.

Ransomware: Stage 3 — Credential Access & Lateral Movement|Mimikatz lsass dump (Sysmon EID 10). PsExec/WMI на другие хосты (EID 7045, EID 4624 LT=3). Масштабирование до DA. Детект: Sysmon EID 10 → lsass, EID 7045 (PSEXESVC), EID 5140 (ADMIN$), множественные 4624 с одного хоста. Это самое долгое окно для детекта.

Ransomware: Stage 4 — Pre-Encryption Actions|vssadmin delete shadows /all /quiet → удаление теневых копий. bcdedit /set {default} recoveryenabled No. wbadmin delete catalog. Детект: EID 4688/Sysmon EID 1: vssadmin.exe delete, bcdedit.exe /set. КРИТИЧНО: немедленная эскалация при детекте vssadmin delete! Изолировать хост ДО начала шифрования.

Ransomware: Stage 5 — Encryption|Массовое обращение к файлам → переименование в .encrypted/.locked/.ryuk. Sysmon EID 11: массовые FileCreate/FileRename. EID 4663: массовый доступ к файлам. CPU нагрузка 100%. Детект с помощью EDR: behavioural signature "mass file rename". Файлы-ловушки (honeypot files): первое переименование honeypot = немедленный алерт.

Ransomware: Stage 6 — Exfiltration (double extortion)|До шифрования: rclone/curl POST больших объёмов данных. NetFlow: аномальный исходящий трафик к облачным провайдерам. Sysmon EID 3: rclone.exe или архиватор → внешний IP. Детект: большой upload в нерабочее время, создание ZIP/RAR архивов перед exfil.

---

## ══ ПРИОРИТЕТ АЛЕРТОВ — МАТРИЦА ══

Как приоритизировать алерты: критический|НЕМЕДЛЕННО (изоляция хоста): vssadmin delete shadows, EID 4662 DCSync от не-DC, EID 10 → lsass.exe, w3wp.exe → cmd.exe, сервис PSEXESVC в нерабочее время, Golden Ticket аномалии, массовое переименование файлов (ransomware).

Как приоритизировать алерты: высокий|Эскалировать в течение часа: EID 4769 RC4 в AES-среде (Kerberoasting), EID 4625 брутфорс + успешный 4624, LSASS доступ от нестандартного процесса, RDP из нового внешнего IP ночью, горизонтальное сканирование из внутренней сети, новый сервис 7045 из нестандартного пути.

Как приоритизировать алерты: средний|Расследовать в течение смены: PowerShell -EncodedCommand от рабочей станции, certutil -urlcache, новый scheduled task с нестандартной командой, добавление пользователя в Admin группу, большой POST к внешнему IP, SSH брутфорс без успешного входа.

Как определить False Positive быстро|1. Есть ли изменение контекста? Легитимный sysadmin работает vs 2 часа ночи. 2. Пользователь подтверждает действие? 3. IP/домен в whitelist (patch servers, AV, backup)? 4. Повторяется по расписанию (automation)? 5. Базовая линия (baseline): это уже видели 100 раз? → FP. Документировать исключение с датой/обоснованием.

---

## ══ ACTIVE DIRECTORY — РАЗВЕДКА ══

AD Enumeration: что ищет атакующий|1. Пользователи домена (net user /domain). 2. Группы и члены (net group "Domain Admins" /domain). 3. DC (nltest /dclist). 4. Политики паролей (net accounts /domain). 5. SPN для Kerberoasting (setspn -Q */*). 6. AS-REP Roastable учётки (PreAuth disabled). 7. Пути к DA (BloodHound).

Разведка AD через LDAP — детект|EID 1644 на DC: LDAP дорогой запрос (expensive search). Wireshark: порт 389/636, фильтр LDAP SearchRequest с широкими критериями (objectClass=*). Sysmon EID 3: ldap запросы с рабочих станций к DC вне AD-интегрированных приложений. Аномалия: >100 LDAP запросов от пользователя за минуту = SharpHound/enum.

Unconstrained Delegation — детект и эксплойт|Компьютер с Unconstrained Delegation хранит TGT всех пользователей которые к нему подключаются. Атакующий (с Admin правами на этой машине) извлекает TGT → pass-the-ticket. Поиск: Get-ADComputer -Filter {TrustedForDelegation -eq $true}. Детект: EID 4769 (TGT запрос к Unconstrained machine), Mimikatz sekurlsa::tickets на этой машине.

Constrained Delegation Abuse — механика|Сервис с Constrained Delegation может получать TGS от имени любого пользователя к указанным SPN (S4U2Self + S4U2Proxy). Атакующий компрометирует сервисную учётку с Constrained Delegation → получает TGS к целевому SPN как любой пользователь (даже DA). Детект: нетипичные S4U2Self TGS запросы, EID 4769 с необычными полями.

Resource-Based Constrained Delegation (RBCD) — атака|Если атакующий имеет write права на атрибут msDS-AllowedToActOnBehalfOfOtherIdentity объекта → может настроить делегирование → получить TGS как Administrator к этому хосту. Вектор: WriteDACL / GenericAll → RBCD → получение SYSTEM. Детект: EID 5136 (изменение атрибута AD объекта), запись в msDS-AllowedToActOnBehalf.

---

## ══ CREDENTIAL DUMPING ══

Credential Dumping: LSASS — методы|1. Mimikatz sekurlsa::logonpasswords (прямое чтение). 2. Task Manager: создать дамп lsass.exe → offline парсинг. 3. comsvcs.dll: rundll32.exe C:\windows\system32\comsvcs.dll MiniDump <PID> lsass.dmp full. 4. Procdump: procdump.exe -ma lsass.exe lsass.dmp. 5. Shadow Copy: извлечь ntds.dit с копии. 6. PPL (Protected Process Light) bypass.

Credential Dumping LSASS — детект|Sysmon EID 10: SourceImage запрашивает TargetImage=lsass.exe с GrantedAccess содержащим 0x10 (PROCESS_VM_READ). EID 1: procdump.exe / rundll32.exe + comsvcs.dll. EID 11: создание .dmp файла. Windows Defender: LSASS credential dumping alert. EDR: PPL + Credential Guard в Windows 11 блокирует большинство методов.

SAM Database Dump — методы и детект|SAM (Security Account Manager) хранит локальные учётки. Dump: reg save HKLM\SAM sam.hive, Mimikatz lsadump::sam. Требует SYSTEM. Детект: EID 4688: reg.exe save с SAM, EID 4663 (доступ к объекту): SAM файл. Sysmon EID 12/11: создание .hive файлов. VSS: эвристика доступа к SAM через shadow copy.

NTDS.dit Dump (DCSync alternative) — механика|ntds.dit — база данных AD с хешами всех пользователей. Метод 1: копировать ntds.dit из работающей системы (Volume Shadow Copy). Метод 2: ntdsutil. Требует доступа к DC + Admin права. Детект: EID 4688: vssadmin.exe create shadow / ntdsutil.exe snapshot. EID 4663: доступ к ntds.dit файлу. Это равнозначно DCSync по серьёзности.

Responder — захват хешей|Responder.py отвечает на LLMNR/NBT-NS/MDNS broadcast → жертва отправляет NTLMv1/v2 хеши. Хеши → hashcat -m 5600. Детект: LLMNR ответы от нелегитимного IP (IDS правило), EID 4776 с ошибками аутентификации от разных пользователей к одному IP (Responder сервер), NetworkCapture: незапрошенные LLMNR ответы.

---

## ══ C2 ДЕТЕКТ ══

Beaconing Detection — как в SIEM|1. Рассчитать стандартное отклонение интервалов между соединениями к одному IP. 2. Если σ < threshold → beaconing (регулярные интервалы). 3. Rita tool: автоматический beaconing score. 4. SIEM: сгруппировать соединения по src_ip + dst_ip + dst_port → percentile интервалов. 5. Дополнительно: байты/пакеты < пороге (маяк отправляет мало данных).

JA3/JA4 Fingerprinting — как использовать|JA3 = MD5 от TLS параметров: version + ciphers + extensions + curves + formats. Каждый клиент/malware имеет уникальный JA3. База: ja3er.com, abuse.ch TLS fingerprints. SIEM: аномальный JA3 от процесса не-браузера (svchost.exe с JA3 отличным от Windows TLS). Suricata: alert tls any any -> any any (ja3_hash; content:"известный_C2_хеш").

Cobalt Strike Beacon — специфические IoC|1. Checksum8: URI заканчивается на 4-байтный checksum (характерный паттерн HTTP). 2. Named pipes: \\.\pipe\MSSE-xxxx-server (случайные символы). 3. Default sleep: 60 сек ± 20% jitter. 4. JA3: характерные хеши по умолчанию. 5. PE: характерный export SSLBeaconAPI. 6. Аномальный User-Agent сочетается с нестандартным контентом.

DNS C2 — детект сигналы|1. NXDomain спам: >50% DNS запросов к домену = NXDOMAIN → DGA или неправильный beacon. 2. Длина имени: avg поддомен >30 символов = кодирование данных. 3. Тип запроса: TXT, CNAME с высокой частотой = нетипично. 4. Объём: DNS трафик от хоста >> baseline (норма: единицы kB/hour). 5. Период: запросы каждые N секунд (cron-like).

---

## ══ СЕТИ — ФИЛЬТРЫ ДЕТЕКТА ══

Wireshark фильтры для атак|ARP Spoofing: arp.duplicate-address-detected. SYN Flood: tcp.flags.syn==1 && tcp.flags.ack==0. DNS Tunneling: dns && frame.len>200. Kerberos RC4: kerberos.etype==23. SMB lateral: smb2.cmd==5 (Create) с путём ADMIN$. NTLM relay: ntlmssp.auth. Beaconing: ip.addr==C2 && tcp.flags.push==1.

SIEM запросы — типовые паттерны|Kerberoasting: EID=4769 TicketEncryptionType=0x17 | stats count by RequestorName, ServiceName | where count>5. PtH: EID=4624 LogonType=3 AuthPackage=NTLM TargetUserName=*admin* | stats count by SourceIP,TargetUserName. Brute: EID=4625 | bucket span=5m | stats count by SourceIP | where count>20.

NetFlow паттерны атак|Горизонтальное сканирование: 1 SRC → много DST, порт фиксированный (445), pps высокий. Lateral movement PsExec: 1 SRC → 1 DST, порт 445, объём ~100KB (копирование exe). DNS tunneling: 1 SRC → 1 DST (DNS server), порт 53, объём аномально высокий. C2 beaconing: 1 SRC → 1 DST, порт 443/80, регулярные интервалы, малый объём.

---

## ══ MULTI-LEVEL DETECTION CHEATSHEET ══

Атака: Brute Force SSH — детект на всех уровнях|L4/NetFlow: множественные SYN к порту 22 с одного IP. L7/Auth.log: "Failed password for X from IP port Y" повторно. SIEM: порог >10 failed за 1 мин с одного IP. Хост: auditd execve sshd. EDR: аномальное количество auth событий. Защита: fail2ban, AllowUsers, ключи. Успешный вход после цепочки = КРИТИЧНО.

Атака: RDP Brute Force — детект на всех уровнях|L4/Firewall: SYN к порту 3389 с внешнего IP. L7/EventLog: EID 4625 LogonType=10 с Foreign IP, много попыток. L7/TermServices log: EID 1149 (successful auth). SIEM: >5 EID 4625 за 1 мин от 1 IP → алерт. EDR: аномальный процесс после RDP входа. NLA (Network Level Auth) снижает риск, но не устраняет брутфорс.

Атака: WebShell — детект на всех уровнях|L7/Web-логи: POST к *.php/*.aspx/*.jsp в upload директории → 200 OK. Затем GET/POST с ?cmd= параметром. L7+Host/Sysmon: w3wp.exe → cmd.exe (ГЛАВНЫЙ ПРИЗНАК). L7/WAF: блок подозрительных параметров. EDR: исполняемый код запущен от IIS пользователя. FIM: изменение файлов в webroot. AV: сигнатура webshell.

Атака: Data Exfiltration через HTTPS — детект|L4/NetFlow: аномальный исходящий объём с хоста. L7/Proxy: большие POST к нетипичному домену, нестандартный User-Agent. L7/DNS: resolver query к домену прямо перед exfil (первое обращение). Хост/Sysmon EID 3: process → external IP. EDR: процесс читает много файлов → потом сетевое соединение. DLP: SSL inspection + content inspection.

Атака: Mimikatz (lsass dump) — детект на всех уровнях|Хост/Sysmon EID 10: процесс → lsass.exe с VM_READ. Хост/EDR: поведенческое: MiniDump API вызов к lsass. AV/EDR: сигнатура mimikatz.exe (или обфусцированная версия = поведенческий). Сеть/Последствия: NTLM аутентификации с несколькими DA учётками с одного IP (использование дампа). SIEM: цепочка EID 10 (lsass) → EID 4624 LogonType=3 DA account.
