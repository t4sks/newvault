---
tags: [soc, mitre, killchain, roadmap]
section: 08
---

# 08 - MITRE ATT&CK и Kill Chain

[[00 - SOC Junior Roadmap (BiZone)]]

> Это каркас мышления SOC-аналитика. Учить первым после баз ОС.

## Cyber Kill Chain (Lockheed Martin)

- [ ] 1. Reconnaissance — разведка
- [ ] 2. Weaponization — создание нагрузки
- [ ] 3. Delivery — доставка (фишинг, USB и т.д.)
- [ ] 4. Exploitation — эксплуатация уязвимости
- [ ] 5. Installation — закрепление (см. persistence)
- [ ] 6. Command & Control — связь с C2
- [ ] 7. Actions on Objectives — достижение цели (эксфильтрация, шифрование)
- [ ] Понимать: на каждом этапе свои детекты и точки разрыва цепочки

## MITRE ATT&CK

- [ ] Что это: база знаний TTP (Tactics, Techniques, Procedures) реальных атак
- [ ] Отличие Tactic / Technique / Sub-technique / Procedure
- [ ] Матрицы: Enterprise, Mobile, ICS
- [ ] ID-нотация: TA000x (тактика), Txxxx (техника), Txxxx.xxx (саб-техника)

### Тактики Enterprise (выучить порядок и смысл)

- [ ] TA0043 Reconnaissance
- [ ] TA0042 Resource Development
- [ ] TA0001 Initial Access (напр. T1566 Phishing)
- [ ] TA0002 Execution (T1059 Command/Scripting Interpreter)
- [ ] TA0003 Persistence (T1053 Scheduled Task, T1547 Boot/Logon Autostart)
- [ ] TA0004 Privilege Escalation
- [ ] TA0005 Defense Evasion (T1027 Obfuscation, T1070 Indicator Removal)
- [ ] TA0006 Credential Access (T1003 OS Credential Dumping)
- [ ] TA0007 Discovery
- [ ] TA0008 Lateral Movement (T1021 Remote Services)
- [ ] TA0009 Collection
- [ ] TA0011 Command and Control (T1071, T1572 Tunneling)
- [ ] TA0010 Exfiltration
- [ ] TA0040 Impact (T1486 Ransomware)

## Связки техника → детект (тренировать)

- [ ] T1003.001 LSASS dump → Sysmon EID 10 доступ к lsass
- [ ] T1053.005 Scheduled Task → Event 4698 / Sysmon
- [ ] T1059.001 PowerShell → Event 4104
- [ ] T1547.001 Run keys → Sysmon EID 13 (registry)
- [ ] T1021.001 RDP → Event 4624 type 10
- [ ] T1486 Ransomware → массовое изменение файлов

## Сопутствующее

- [ ] MITRE D3FEND (контрмеры)
- [ ] Detection engineering: Sigma-правила, маппинг на ATT&CK
- [ ] Покрытие детектами (ATT&CK Navigator — тепловая карта)

## Progress

- [ ] Раздел MITRE / Kill Chain пройден полностью

---

## 🧪 Тест для повторения

> [!question]- 1. Перечисли 7 этапов Cyber Kill Chain.
> Reconnaissance → Weaponization → Delivery → Exploitation → Installation → Command & Control → Actions on Objectives. На каждом этапе свои детекты; цель защиты — разорвать цепочку как можно раньше.

> [!question]- 2. Разница между Tactic, Technique, Procedure в ATT&CK.
> Tactic — «зачем» (цель этапа, напр. Persistence, TA0003). Technique — «как» (способ достижения, напр. T1053 Scheduled Task). Procedure — конкретная реализация техники конкретным актором/инструментом.

> [!question]- 3. К какой тактике относится OS Credential Dumping и какой её ID?
> Tactic: Credential Access (TA0006). Technique: T1003 (OS Credential Dumping), саб-техника LSASS Memory — T1003.001, DCSync — T1003.006.

> [!question]- 4. Сопоставь технику T1059.001 с детектом.
> T1059.001 — PowerShell (тактика Execution). Детект: Event 4104 (ScriptBlock logging), 4103 (module), Sysmon EID 1 на powershell.exe с подозрительными аргументами (-enc, IEX, DownloadString).

> [!question]- 5. Зачем SOC-аналитику ATT&CK Navigator?
> Визуализация покрытия детектами по матрице (тепловая карта): видно, какие техники закрыты правилами, а какие — слепые зоны. Помогает приоритизировать разработку детектов и оценивать зрелость мониторинга.

> [!question]- 6. Что такое Sigma и как связана с ATT&CK?
> Sigma — открытый формат правил детекта для логов (конвертируется под разные SIEM). Правила маркируются тегами ATT&CK (techniqueID), что позволяет маппить детекты на матрицу и измерять покрытие.

---

## 📚 Источники для подготовки

**MITRE ATT&CK**
- Энциклопедия Касперского — Что такое MITRE ATT&CK (тактики/техники, матрицы): https://encyclopedia.kaspersky.ru/glossary/mitre-attack/
- SecurityVision — MITRE в кибербезопасности: матрица ATT&CK, CVE, CWE, стратегии SOC: https://www.securityvision.ru/info/mitre/
- Blue Team Cookbook — MITRE ATT&CK, Cyber Kill Chain, UKC (для blue team / SOC, читать целиком): https://vasilisa-l.gitbook.io/blue-team-cookbook/soc/mitre-attack
- Habr — MITRE ATT&CK: обзор тактик разведки (как читать матрицу на примере): https://habr.com/ru/articles/954656/

**Cyber Kill Chain**
- Habr (Panda) — Что такое Cyber Kill Chain и зачем в стратегии защиты: https://habr.com/ru/company/panda/blog/327488/
- Habr (SecurityVision) — Как научиться выстраивать килчейн (7 этапов): https://habr.com/ru/companies/securityvison/articles/777790/
- Habr — Атака как кейс: ATT&CK + D3FEND + Kill Chain + CVSS (сквозной пример): https://habr.com/ru/articles/909562/

**Официальные базы**
- MITRE ATT&CK: https://attack.mitre.org/
- ATT&CK Navigator (карта покрытия детектами): https://mitre-attack.github.io/attack-navigator/
- Sigma rules (детект-правила с маппингом на ATT&CK): https://github.com/SigmaHQ/sigma
- Mindmap от друзей: https://xmind.app/m/WwtB/#
