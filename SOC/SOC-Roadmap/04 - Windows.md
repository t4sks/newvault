---
tags: [soc, windows, registry, roadmap]
section: 04
---

# 04 - Windows

[[00 - SOC Junior Roadmap (BiZone)]]

## Реестр Windows

- [ ] Структура: ульи (hives), ключи, значения, типы (REG_SZ, REG_DWORD, REG_BINARY)
- [ ] HKLM (HKEY_LOCAL_MACHINE) — настройки всей машины
- [ ] HKCU (HKEY_CURRENT_USER) — настройки текущего пользователя
- [ ] HKCR, HKU, HKCC

### Конкретные ульи (где лежат файлы)

- [ ] **SAM** — `C:\Windows\System32\config\SAM` → локальные учётки и хэши паролей
- [ ] **SECURITY** — `...\config\SECURITY` → политики, LSA-секреты
- [ ] **SOFTWARE** — `...\config\SOFTWARE` → установленный софт, настройки
- [ ] **SYSTEM** — `...\config\SYSTEM` → драйверы, службы, boot-ключ (нужен для расшифровки SAM)
- [ ] **HKCU** (NTUSER.DAT) — в профиле пользователя `C:\Users\<user>\NTUSER.DAT`

## Дамп кредов в Windows (атака + детект)

- [ ] LSASS memory dump (хэши, plaintext) — mimikatz `sekurlsa::logonpasswords`
  - [ ] Детект: доступ к процессу lsass.exe (Event 4656/4663, Sysmon EID 10)
- [ ] Дамп SAM + SYSTEM (offline) → расшифровка хэшей
- [ ] DCSync (через AD-репликацию) — mimikatz `lsadump::dcsync`
  - [ ] Детект: репликационные запросы не от DC (Event 4662)
- [ ] LSA secrets, cached credentials
- [ ] Mimikatz — ознакомиться с основными модулями (sekurlsa, lsadump, kerberos)
- [ ] Альтернативы: procdump + mimikatz offline, comsvcs.dll MiniDump, secretsdump.py (impacket)

## Порядок запуска процессов в Windows

- [ ] Boot → System (PID 4) → smss.exe → csrss.exe, wininit.exe
- [ ] wininit.exe → services.exe, lsass.exe, lsm
- [ ] winlogon.exe → userinit.exe → explorer.exe
- [ ] services.exe → svchost.exe (множество)
- [ ] **Зачем:** знать нормальное дерево, чтобы видеть аномалии (lsass с дочерним cmd.exe = подозрительно; svchost не от services.exe = подозрительно)
- [ ] Легитимные родитель-потомок связи (baseline)

## Командная строка: cmd и PowerShell

- [ ] cmd основы, разница с PowerShell
- [ ] PowerShell: cmdlet (Verb-Noun, напр. `Get-Process`), pipeline объектов
- [ ] **AMSI** (Antimalware Scan Interface) — что это, как сканирует скрипты, методы обхода (концептуально)
- [ ] Скачивание нагрузки через PS: `Invoke-WebRequest`, `Net.WebClient.DownloadString`, `IEX`
- [ ] Запуск обфусцированного кода: `IEX`, `-EncodedCommand`, `FromBase64String`
- [ ] Виды обфускации: base64, gzip/inflate (Deflate), string concat, char codes
- [ ] Детект: подозрительные PS-команды в Event 4104 (ScriptBlock logging), `-enc`, длинные base64

## Логи Windows

- [ ] Event Viewer, EVTX-файлы (`C:\Windows\System32\winevt\Logs\`)
- [ ] Security log — ключевые EventID:
  - [ ] 4624 успешный вход / 4625 неудачный
  - [ ] 4672 special privileges (админ-вход)
  - [ ] 4688 создание процесса (+ command line)
  - [ ] 4720/4726 создание/удаление учётки
  - [ ] 4662 операция с AD-объектом (DCSync детект)
  - [ ] 4768/4769 Kerberos TGT/TGS (детект Kerberoasting)
- [ ] System, Application logs
- [ ] PowerShell logs: 4103 (module), 4104 (scriptblock)
- [ ] **Sysmon** — must have: EID 1 (process), 3 (network), 7 (image load), 8 (CreateRemoteThread), 10 (process access), 11 (file create), 13 (registry)
- [ ] Логон-типы (2 интерактивный, 3 сеть, 10 RDP, 9 NewCredentials)

## Persistence в Windows (атака + детект)

- [ ] Run/RunOnce ключи реестра (HKLM/HKCU `...\CurrentVersion\Run`)
- [ ] Scheduled Tasks (`schtasks`, Task Scheduler) → Event 4698
- [ ] Services (новая служба) → Event 7045
- [ ] WMI event subscriptions
- [ ] Startup folder
- [ ] DLL hijacking / search order
- [ ] Logon scripts, GPO
- [ ] Accessibility features (sethc.exe sticky keys)

## Аутентификация Windows

- [ ] NTLM (challenge-response), NTLMv1/v2
- [ ] Kerberos (см. [[09 - Active Directory и атаки]])
- [ ] LM (устаревший)
- [ ] Pass-the-Hash, Pass-the-Ticket (концепция)

## Progress

- [ ] Раздел Windows пройден полностью

---

## 🧪 Тест для повторения

> [!question]- 1. Назови основные ульи реестра, где лежат файлы и что содержат.
> SAM (`...\config\SAM`) — локальные учётки и NT-хэши. SECURITY — LSA-секреты, политики. SOFTWARE — установленный софт. SYSTEM — службы, драйверы, boot key (нужен для расшифровки SAM). HKCU = NTUSER.DAT в профиле пользователя. Ключи автозапуска: HKLM/HKCU `...\CurrentVersion\Run`.

> [!question]- 2. Как дампят креды в Windows (3 способа) и как детектить дамп LSASS?
> Дамп памяти LSASS (mimikatz `sekurlsa::logonpasswords`, procdump, comsvcs MiniDump), offline-дамп SAM+SYSTEM, DCSync через репликацию AD. Детект LSASS: обращение к процессу lsass.exe — Sysmon EID 10 (ProcessAccess), Security 4656/4663.

> [!question]- 3. Почему важно знать порядок запуска процессов? Приведи аномалию.
> Чтобы отличать норму от аномалии по родитель-потомок. Норма: services.exe → svchost.exe; winlogon → userinit → explorer. Аномалия: lsass.exe порождает cmd.exe/powershell.exe; svchost.exe не от services.exe; Office-приложение (winword.exe) порождает powershell.exe.

> [!question]- 4. Что такое AMSI и при чём тут обфускация PowerShell?
> AMSI (Antimalware Scan Interface) — интерфейс, через который AV/EDR сканируют содержимое скриптов/команд перед выполнением. Обфускация (base64, Deflate/inflate, конкатенация строк, char-коды) и обход AMSI используются, чтобы вредоносный PS не распознался. Детект: Event 4104 (ScriptBlock), флаги `-enc`/`-EncodedCommand`, длинные base64-строки.

> [!question]- 5. Ключевые EventID Security-лога для SOC.
> 4624 вход / 4625 неуспех / 4672 админ-привилегии / 4688 создание процесса (+cmdline) / 4720 создание учётки / 4698 scheduled task / 4662 операция с AD-объектом (DCSync) / 4768-4769 Kerberos. Logon type: 2 интерактивный, 3 сетевой, 10 RDP.

> [!question]- 6. Где хранятся логи Windows и что добавляет Sysmon?
> EVTX в `C:\Windows\System32\winevt\Logs\` (Security/System/Application + PowerShell). Sysmon добавляет детальную телеметрию: EID 1 (process create + хэши), 3 (network), 7 (image load), 8 (CreateRemoteThread), 10 (process access), 11 (file create), 13 (registry). Без Sysmon многое не логируется.

> [!question]- 7. Виды persistence в Windows (минимум 5).
> Run/RunOnce ключи, Scheduled Tasks (4698), новая Service (7045), WMI event subscription, Startup folder, DLL hijacking, accessibility (sethc.exe), logon scripts/GPO.

---

## 📚 Источники для подготовки

**Реестр Windows (SAM / SECURITY / SOFTWARE / SYSTEM)**
- Habr — Автоматизация выявления вредоноса в реестре (где лежат ульи, пути к файлам): https://habr.com/ru/articles/771742/
- Habr — Анализ активности пользователей Windows (SAM, Regedit, артефакты): https://habr.com/ru/articles/927456/
- Habr — Хранение и шифрование паролей Windows (LM/NT, V-блок в SAM): https://habr.com/ru/post/114150/
- HackWare — Анализ реестра Windows (что такое куст/улей): https://hackware.ru/?p=14371

**Дамп кредов / mimikatz + детект LSASS**
- Habr (Angara) — Дампы LSASS для всех (mimikatz, sekurlsa, методы): https://habr.com/ru/companies/angarasecurity/articles/661341/
- Habr (Angara) — Детектирование дампа памяти LSASS. SOC наносит ответный удар (детект, EID 4656/10): https://habr.com/ru/companies/angarasecurity/articles/679592/
- winitpro — Извлекаем пароли/хэши через mimikatz (модули, команды): https://winitpro.ru/index.php/2013/12/24/poluchenie-v-otkrytom-vide-parolej-polzovatelej-avtorizovannyx-v-windows/

**Порядок запуска процессов / легитимные процессы**
- Habr — Галопом по Европам: легитимные процессы Windows на пальцах (smss, csrss, lsass, svchost): https://habr.com/ru/articles/784960/
- ivanovds.ru — Основные процессы Windows (дерево, родитель-потомок): https://ivanovds.ru/Windows/windows_core_processes/

**AMSI / PowerShell / обфускация + детект**
- Habr — AMSI bypass: от истоков к Windows 11 (как работает AMSI): https://habr.com/ru/articles/758550/
- Habr — BypassAV, бесфайловая атака и AMSI (теория): https://habr.com/ru/articles/755034/
- Habr (Microsoft) — Теперь я тебя вижу: выявление бесфайловых вредоносов (base64 в реестре): https://habr.com/ru/company/microsoft/blog/352376/

**Логи Windows / Sysmon / EventID**
- Habr (Microsoft) — Sysmon для безопасника: расширяем аудит событий: https://habr.com/ru/company/microsoft/blog/352692/
- Habr — Профилируем события Sysmon при внедрении (для инженера SOC): https://habr.com/ru/articles/664916/
- HackMD — Журналы событий Windows / Sysmon (шпаргалка по EID): https://hackmd.io/@hackermans/BkFx7kx8T
- Sysmon-config (рекомендуемые конфиги SwiftOnSecurity / Olaf Hartong): https://github.com/SwiftOnSecurity/sysmon-config

**Persistence в Windows + детект**
- Энциклопедия Касперского — Что такое persistence (обзор методов): https://encyclopedia.kaspersky.ru/glossary/persistence/
- Инфобезопасность — Как исследуют реестр и журналы при расследовании (Run/RunOnce, Services, WMI, ShimCache/Amcache): https://infobezopasnost.ru/blog/articles/kak-issleduyut-reestr-i-zhurnaly-sobytij-windows-pri-rassledovanii/

**Справочник EventID**
- Ultimate Windows Security — Security Log Encyclopedia: https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/
- LOLBAS (легитимные бинари Windows, используемые для abuse): https://lolbas-project.github.io/
