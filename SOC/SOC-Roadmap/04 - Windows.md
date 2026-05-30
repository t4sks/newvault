---
tags: [soc, windows, registry, roadmap]
section: 04
---

# 04 - Windows

[[00 - SOC Junior Roadmap (BiZone)]]

> **Лаба.** Заведи ВМ **Windows 10/11** (или Windows Server 2022 — eval-версия бесплатна 180 дней с сайта Microsoft Evaluation Center). Поставь **Sysinternals Suite** (Process Explorer, Procmon, Autoruns) и **Sysmon** с конфигом SwiftOnSecurity. Snapshot чистой системы — обязательно. Для AD-практики Windows Server понадобится в [[09 - Active Directory и атаки]].
>
> **Формат:** 📖 читать · 🛠 трогать руками · 💬 промпт мне.

---

## Реестр Windows

### Структура реестра
- [ ] Ульи (hives), ключи, значения, типы (REG_SZ, REG_DWORD, REG_BINARY, REG_MULTI_SZ)
- [ ] HKLM, HKCU, HKCR, HKU, HKCC — что где

📖 **Читать:** HackWare «Анализ реестра Windows» (что такое куст/улей) https://hackware.ru/?p=14371 ; Habr «Активность пользователей Windows» https://habr.com/ru/articles/927456/

🛠 **Потрогать:**
```cmd
regedit                              :: открой, полазь по HKLM и HKCU
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"
reg add HKCU\Software\Test /v MyVal /t REG_SZ /d hello
reg query HKCU\Software\Test
reg delete HKCU\Software\Test /f
```
Задание: найди в реестре все программы автозапуска (Run/RunOnce в HKLM и HKCU), запиши пути к ключам.

💬 **Промпт мне:** «Объясни разницу HKLM vs HKCU на примере: где живут настройки всей машины, а где — конкретного юзера, и почему это важно для форензики (NTUSER.DAT)».

### Конкретные ульи на диске
- [ ] **SAM** — `C:\Windows\System32\config\SAM` → локальные учётки и NT-хеши
- [ ] **SECURITY** → LSA-секреты, политики
- [ ] **SOFTWARE** → установленный софт
- [ ] **SYSTEM** → службы, драйверы, boot key (нужен для расшифровки SAM)
- [ ] **HKCU = NTUSER.DAT** в профиле пользователя

📖 **Читать:** Habr «Автоматизация выявления вредоноса в реестре» (точные пути ульев) https://habr.com/ru/articles/771742/ ; «Хранение и шифрование паролей Windows» https://habr.com/ru/post/114150/

🛠 **Потрогать:**
```cmd
dir C:\Windows\System32\config\          :: увидишь SAM, SECURITY, SOFTWARE, SYSTEM
dir C:\Users\%USERNAME%\NTUSER.DAT /a    :: твой улей HKCU
:: попробуй открыть SAM напрямую — увидишь "access denied" (его держит система)
```
Задание: объясни себе, почему SAM нельзя просто скопировать на работающей системе и зачем атакующему нужен ещё и SYSTEM-улей (ответ: bootkey).

💬 **Промпт мне:** «Я хочу понять цепочку: атакующий хочет хеши паролей. Какие файлы ему нужны, почему два (SAM+SYSTEM), и как это делается offline vs из памяти LSASS? Разложи по шагам».

---

## Дамп кредов в Windows (атака + детект)

- [ ] LSASS memory dump (mimikatz `sekurlsa::logonpasswords`) → детект: Sysmon EID 10 / Security 4656/4663
- [ ] Дамп SAM+SYSTEM (offline)
- [ ] DCSync (`lsadump::dcsync`) → детект: Event 4662 (см. [[09 - Active Directory и атаки]])
- [ ] LSA secrets, cached credentials
- [ ] mimikatz — основные модули (sekurlsa, lsadump, kerberos)
- [ ] Альтернативы: procdump, comsvcs.dll MiniDump, secretsdump.py (impacket)

📖 **Читать:** Habr (Angara) «Дампы LSASS для всех» (атака) https://habr.com/ru/companies/angarasecurity/articles/661341/ ; «Детектирование дампа LSASS. SOC наносит ответный удар» (детект!) https://habr.com/ru/companies/angarasecurity/articles/679592/

🛠 **Потрогать (в изолированной ВМ, snapshot обязателен):**
```cmd
:: Способ дампа без mimikatz — встроенный comsvcs (для понимания атаки):
tasklist | findstr lsass                  :: найди PID lsass
:: rundll32 C:\Windows\System32\comsvcs.dll, MiniDump <PID> C:\temp\l.dmp full
:: ^ это создаст дамп памяти LSASS — антивирус среагирует, это нормально

:: ДЕТЕКТ-сторона (главное для SOC):
:: в Event Viewer → Sysmon/Operational ищи EID 10 с TargetImage lsass.exe
```
Задание: включи Sysmon, попробуй обратиться к процессу lsass (хоть через Process Explorer → правый клик → Properties), найди в Sysmon EID 10 событие доступа к lsass.exe.

💬 **Промпт мне:** «Покажи, как настроить детект доступа к LSASS: какое Sysmon-событие (EID 10), какие поля смотреть (GrantedAccess 0x1010/0x1410), и как отличить легитимный доступ (антивирус) от mimikatz».

---

## Порядок запуска процессов Windows

- [ ] System (PID 4) → smss.exe → csrss.exe, wininit.exe
- [ ] wininit.exe → services.exe, lsass.exe
- [ ] winlogon.exe → userinit.exe → explorer.exe
- [ ] services.exe → svchost.exe
- [ ] **Зачем:** baseline нормального дерева, чтобы видеть аномалии
- [ ] Легитимные связи родитель-потомок

📖 **Читать:** Habr «Легитимные процессы Windows на пальцах» (must-read) https://habr.com/ru/articles/784960/ ; «Основные процессы Windows» https://ivanovds.ru/Windows/windows_core_processes/

🛠 **Потрогать (Process Explorer из Sysinternals):**
```
1. Открой Process Explorer — увидишь дерево процессов
2. Найди: System(4) → smss → csrss/wininit → services → svchost
3. Проверь: explorer.exe — кто родитель? (должен быть userinit, потом он исчезает)
4. Запусти cmd.exe из explorer — увидишь связь explorer→cmd
5. Аномалия для тренировки: запусти powershell ИЗ winword (если есть Office) —
   увидишь winword→powershell, классический признак фишинга с макросом
```
Задание: запиши «нормальное» дерево по памяти. Потом в Process Explorer найди процесс, у которого «неправильный» родитель, и объясни, почему это было бы подозрительно в реальной системе.

💬 **Промпт мне:** «Дай таблицу нормальных родитель→потомок связей ключевых процессов Windows и список из 7 аномалий (типа lsass→cmd, svchost не от services), которые SOC-аналитик должен сразу замечать».

---

## Командная строка: cmd и PowerShell

- [ ] cmd основы, отличие от PowerShell
- [ ] PowerShell: cmdlet (Verb-Noun), pipeline объектов
- [ ] **AMSI** — что это, как сканирует, концепция обхода
- [ ] Скачивание нагрузки через PS: `Invoke-WebRequest`, `Net.WebClient.DownloadString`, `IEX`
- [ ] Запуск обфусцированного кода: `IEX`, `-EncodedCommand`, `FromBase64String`
- [ ] Виды обфускации: base64, gzip/inflate (Deflate), конкатенация, char-коды
- [ ] Детект: Event 4104 (ScriptBlock), `-enc`, длинные base64

📖 **Читать:** Habr «AMSI bypass: от истоков к Windows 11» https://habr.com/ru/articles/758550/ ; «Бесфайловые атаки и AMSI (теория)» https://habr.com/ru/articles/755034/

🛠 **Потрогать (PowerShell):**
```powershell
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5   # pipeline объектов
# base64-обфускация (для понимания детекта):
$cmd = 'Write-Host "hello"'
$enc = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
powershell -EncodedCommand $enc      # запуск закодированного
# декодирование (как делает аналитик):
[Text.Encoding]::Unicode.GetString([Convert]::FromBase64String($enc))
# включи ScriptBlock logging и посмотри Event 4104:
# gpedit → Admin Templates → Windows Components → PowerShell → Turn on Script Block Logging
```
Задание: закодируй команду в base64, запусти через `-enc`, потом найди её в Event Viewer (Microsoft-Windows-PowerShell/Operational, EID 4104) уже в раскодированном виде. Это и есть детект обфускации.

💬 **Промпт мне:** «Объясни, что такое AMSI, на каком этапе оно сканирует PowerShell, и почему Event 4104 (ScriptBlock logging) ловит даже обфусцированный код. Дай признаки вредоносного PS в логах».

---

## Логи Windows

- [ ] Event Viewer, EVTX (`C:\Windows\System32\winevt\Logs\`)
- [ ] Security log — ключевые EventID (4624/4625/4672/4688/4720/4698/4662/4768/4769)
- [ ] System, Application
- [ ] PowerShell logs: 4103 (module), 4104 (scriptblock)
- [ ] **Sysmon** — EID 1/3/7/8/10/11/13
- [ ] Logon-типы (2 интерактивный, 3 сеть, 10 RDP, 9 NewCredentials)

📖 **Читать:** Habr «Sysmon для безопасника» https://habr.com/ru/company/microsoft/blog/352692/ ; HackMD шпаргалка по EID https://hackmd.io/@hackermans/BkFx7kx8T ; справочник https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/

🛠 **Потрогать:**
```
1. Установи Sysmon с конфигом:
   sysmon64 -accepteula -i sysmonconfig-export.xml
   (конфиг: github.com/SwiftOnSecurity/sysmon-config)
2. Event Viewer → Windows Logs → Security:
   - залогинься/разлогинься → найди 4624 (успешный вход) с Logon Type
   - введи неверный пароль → найди 4625
3. Запусти cmd → найди 4688 (создание процесса) с командной строкой
4. Event Viewer → Applications and Services → Microsoft → Windows → Sysmon:
   - EID 1 (создание процесса), EID 3 (сетевое соединение)
```
Задание: сделай RDP-подключение (или симулируй `runas`), найди событие 4624 с Logon Type 10. Запиши все типы логонов и что они значат.

💬 **Промпт мне:** «Дай шпаргалку SOC-аналитика по Windows EventID: для каждого (4624/4625/4672/4688/4698/4720/4768/4769/7045/Sysmon 1/3/10/13) — что значит, на какую атаку указывает, что смотреть в полях».

---

## Persistence в Windows (атака + детект)

- [ ] Run/RunOnce ключи реестра
- [ ] Scheduled Tasks → Event 4698
- [ ] Services (новая служба) → Event 7045
- [ ] WMI event subscriptions
- [ ] Startup folder
- [ ] DLL hijacking / search order
- [ ] Logon scripts, GPO
- [ ] Accessibility (sethc.exe sticky keys)

📖 **Читать:** Энциклопедия Касперского https://encyclopedia.kaspersky.ru/glossary/persistence/ ; Инфобезопасность «Как исследуют реестр и журналы» https://infobezopasnost.ru/blog/articles/kak-issleduyut-reestr-i-zhurnaly-sobytij-windows-pri-rassledovanii/

🛠 **Потрогать (Autoruns из Sysinternals — главный инструмент):**
```cmd
:: поставь закреп через Run-ключ:
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /t REG_SZ /d "calc.exe"
:: поставь scheduled task:
schtasks /create /tn "EvilTask" /tr "calc.exe" /sc minute /mo 1
:: ДЕТЕКТ:
:: 1. открой Autoruns — увидишь свой Backdoor и EvilTask подсвеченными
:: 2. Event Viewer → Security → найди 4698 (создание задачи)
schtasks /query /tn "EvilTask"
:: убери:
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /f
schtasks /delete /tn "EvilTask" /f
```
Задание: поставь 3 разных закрепа (Run-ключ, scheduled task, startup folder), найди все три через Autoruns, потом найди соответствующие события в логах. Откати snapshot.

💬 **Промпт мне:** «Помоги составить чеклист hunting persistence на Windows: какие ключи реестра, какие EventID (4698, 7045), что показывает Autoruns, как отличить легит от закрепа. По каждому пункту — команда проверки».

---

## Аутентификация Windows

- [ ] NTLM (challenge-response), NTLMv1/v2
- [ ] Kerberos (детально в [[09 - Active Directory и атаки]])
- [ ] LM (устаревший)
- [ ] Pass-the-Hash, Pass-the-Ticket (концепция)

📖 **Читать:** Habr (PT) «Погружение в AD» (NTLM, PtH) https://habr.com/ru/companies/pt/articles/423903/

🛠 **Потрогать:**
```powershell
# посмотри, какие протоколы аутентификации использовались:
Get-WinEvent -LogName Security -MaxEvents 50 | Where-Object {$_.Id -eq 4624} |
  Format-Table TimeCreated, @{n='LogonType';e={$_.Properties[8].Value}}
# Logon Type 3 = сетевой (NTLM/Kerberos), 10 = RDP
```
Задание: разберись, чем NTLM challenge-response отличается от Kerberos (билеты). Запиши, почему Pass-the-Hash работает именно с NTLM.

💬 **Промпт мне:** «Сравни NTLM и Kerberos простыми словами: как каждый аутентифицирует, почему Pass-the-Hash возможен для NTLM, и где в логах видно какой протокол использовался».

---

## Progress

- [ ] Раздел Windows пройден полностью
- [ ] Sysmon установлен и настроен
- [ ] Все 🛠 задания выполнены, написан чеклист persistence-hunting

---

## 🔭 Связь со следующими шагами

- **→ Blueteam (твоя цель):** этот раздел — ядро Windows-форензики. EventID + Sysmon + Autoruns + детект LSASS/persistence — то, на чём строится весь threat hunting. Дальше: Sigma-правила, Velociraptor, KAPE, DFIR-курсы.
- **→ Инфрапентест:** дамп LSASS, PtH, persistence — это пост-эксплуатация Windows. Дальше: TryHackMe «Windows PrivEsc», PowerView, winPEAS.
- **→ AD:** аутентификация и LSASS ведут прямо в [[09 - Active Directory и атаки]] — следующий по приоритету раздел.

---

## 🧪 Тест для повторения

> [!question]- 1. Назови основные ульи реестра, где лежат файлы и что содержат.
> SAM (`...\config\SAM`) — локальные учётки и NT-хеши. SECURITY — LSA-секреты, политики. SOFTWARE — установленный софт. SYSTEM — службы, драйверы, boot key (нужен для расшифровки SAM). HKCU = NTUSER.DAT в профиле. Автозапуск — HKLM/HKCU `...\CurrentVersion\Run`.

> [!question]- 2. Как дампят креды в Windows (3 способа) и как детектить дамп LSASS?
> Дамп памяти LSASS (mimikatz sekurlsa, procdump, comsvcs MiniDump), offline-дамп SAM+SYSTEM, DCSync через репликацию AD. Детект LSASS: Sysmon EID 10 (ProcessAccess к lsass.exe), Security 4656/4663, поле GrantedAccess.

> [!question]- 3. Почему важно знать порядок запуска процессов? Дай 3 аномалии.
> Чтобы отличать норму от аномалии по родитель-потомок. Аномалии: lsass→cmd/powershell; svchost не от services.exe; winword→powershell (фишинг с макросом); explorer от неожиданного родителя.

> [!question]- 4. Что такое AMSI и при чём обфускация PowerShell?
> AMSI (Antimalware Scan Interface) — интерфейс, через который AV/EDR сканируют содержимое скриптов перед выполнением. Обфускация (base64, Deflate, конкатенация) и AMSI-bypass используются, чтобы вредоносный PS не распознался. Детект: Event 4104, флаги -enc, длинные base64.

> [!question]- 5. Перечисли ключевые Security EventID и что значат.
> 4624 вход / 4625 неуспех / 4672 админ-привилегии / 4688 создание процесса / 4720 создание учётки / 4698 scheduled task / 4662 операция с AD-объектом / 4768-4769 Kerberos / 7045 новая служба.

> [!question]- 6. Logon-типы в Event 4624 — назови 4 и что значат.
> Type 2 — интерактивный (за клавиатурой), 3 — сетевой (SMB/NTLM/Kerberos), 10 — RemoteInteractive (RDP), 9 — NewCredentials (runas /netonly), 5 — служба, 4 — batch.

> [!question]- 7. Где хранятся логи Windows и что добавляет Sysmon?
> EVTX в `C:\Windows\System32\winevt\Logs\` (Security/System/Application + PowerShell). Sysmon добавляет: EID 1 (process + хеши + cmdline), 3 (network), 7 (image load), 8 (CreateRemoteThread), 10 (process access), 11 (file create), 13 (registry). Без Sysmon многое не логируется.

> [!question]- 8. Виды persistence в Windows (минимум 6) с EventID детекта.
> Run/RunOnce ключи (Sysmon 13), Scheduled Tasks (4698), новая Service (7045), WMI subscription, Startup folder, DLL hijacking, accessibility (sethc.exe), logon scripts/GPO. Инструмент-агрегатор: Autoruns.

> [!question]- 9. Что такое NTLM и почему возможен Pass-the-Hash?
> NTLM — challenge-response протокол: сервер шлёт challenge, клиент отвечает, используя NT-хеш пароля. Сам пароль не передаётся. PtH работает, потому что для аутентификации достаточно хеша — знать сам пароль не нужно.

> [!question]- 10. Где в Windows смотреть выполненные процессы постфактум (форензика)?
> Event 4688 (если включён аудит создания процессов) и Sysmon EID 1 (с cmdline и хешами). Артефакты: Prefetch, Amcache.hve, ShimCache (AppCompatCache) — показывают, что запускалось, даже если файл удалён.

> [!question]- 11. Что такое DLL hijacking?
> Подмена/подкладывание вредоносной DLL туда, где приложение её ищет по порядку поиска (search order), из-за чего грузится злоумышленная библиотека вместо легитимной. Persistence + privesc одновременно. Детект: Sysmon EID 7 (Image Load) из необычных путей.

> [!question]- 12. Чем cmd отличается от PowerShell с точки зрения SOC?
> cmd — текстовый вывод, ограничен. PowerShell — объектный pipeline, полный доступ к .NET и WMI, поэтому мощнее для атак (загрузка в память, fileless). Зато логируется лучше: ScriptBlock (4104) и Module (4103) logging дают полную видимость.

---

## 📚 Все источники раздела (сводно)

**Реестр**
- HackWare — анализ реестра: https://hackware.ru/?p=14371
- Habr — вредонос в реестре (пути ульев): https://habr.com/ru/articles/771742/
- Habr — хранение паролей: https://habr.com/ru/post/114150/

**Дамп кредов + детект**
- Habr (Angara) — дампы LSASS: https://habr.com/ru/companies/angarasecurity/articles/661341/
- Habr (Angara) — детект дампа LSASS: https://habr.com/ru/companies/angarasecurity/articles/679592/

**Процессы**
- Habr — легитимные процессы на пальцах: https://habr.com/ru/articles/784960/
- ivanovds — основные процессы: https://ivanovds.ru/Windows/windows_core_processes/

**AMSI / PowerShell**
- Habr — AMSI bypass: https://habr.com/ru/articles/758550/
- Habr — бесфайловые атаки и AMSI: https://habr.com/ru/articles/755034/

**Логи / Sysmon**
- Habr — Sysmon для безопасника: https://habr.com/ru/company/microsoft/blog/352692/
- HackMD — шпаргалка EID: https://hackmd.io/@hackermans/BkFx7kx8T
- Ultimate Windows Security — справочник: https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/
- Sysmon-config: https://github.com/SwiftOnSecurity/sysmon-config

**Persistence**
- Энциклопедия Касперского: https://encyclopedia.kaspersky.ru/glossary/persistence/
- Инфобезопасность — реестр и журналы: https://infobezopasnost.ru/blog/articles/kak-issleduyut-reestr-i-zhurnaly-sobytij-windows-pri-rassledovanii/

**Инструменты / практика**
- Sysinternals Suite: https://learn.microsoft.com/sysinternals/
- LOLBAS: https://lolbas-project.github.io/
- TryHackMe — Windows Fundamentals: https://tryhackme.com/module/windows-fundamentals
