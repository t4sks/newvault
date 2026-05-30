---
tags: [soc, mitre, killchain, roadmap]
section: 08
---

# 08 - MITRE ATT&CK и Kill Chain

[[00 - SOC Junior Roadmap (BiZone)]]

> **Лаба.** Тут лаба = сайт attack.mitre.org + ATT&CK Navigator (браузер). Практика: бери любую атаку из своих разделов (LSASS-дамп, Kerberoasting) и раскладывай по тактикам/техникам. Это каркас мышления SOC — учи первым после баз ОС.
>
> **Формат:** 📖 читать · 🛠 трогать · 💬 промпт мне.

---

## Cyber Kill Chain (Lockheed Martin)

- [ ] 1. Reconnaissance — разведка
- [ ] 2. Weaponization — создание нагрузки
- [ ] 3. Delivery — доставка (фишинг, USB)
- [ ] 4. Exploitation — эксплуатация
- [ ] 5. Installation — закрепление
- [ ] 6. Command & Control — связь с C2
- [ ] 7. Actions on Objectives — цель (эксфильтрация, шифрование)
- [ ] На каждом этапе — свои детекты и точки разрыва

📖 **Читать:** Habr (Panda) «Что такое Cyber Kill Chain» https://habr.com/ru/company/panda/blog/327488/ ; Habr (SecurityVision) «Как выстраивать килчейн» https://habr.com/ru/companies/securityvison/articles/777790/

🛠 **Потрогать:** Возьми сценарий «фишинг → макрос → PowerShell → C2 → кража данных» и распиши, какой этап Kill Chain соответствует каждому шагу и где его можно разорвать.

💬 **Промпт мне:** «Дай мне реальный сценарий атаки и попроси разложить по 7 этапам Kill Chain. Потом проверь и покажи, где на каждом этапе SOC может разорвать цепочку».

---

## MITRE ATT&CK

- [ ] Что это: база TTP реальных атак
- [ ] Tactic / Technique / Sub-technique / Procedure
- [ ] Матрицы: Enterprise, Mobile, ICS
- [ ] ID: TA000x (тактика), Txxxx (техника), Txxxx.xxx (саб)

📖 **Читать:** Энциклопедия Касперского https://encyclopedia.kaspersky.ru/glossary/mitre-attack/ ; Blue Team Cookbook (MITRE для SOC) https://vasilisa-l.gitbook.io/blue-team-cookbook/soc/mitre-attack ; Habr «Обзор тактик разведки» https://habr.com/ru/articles/954656/

🛠 **Потрогать:**
```
1. Открой attack.mitre.org → выбери технику T1003 (OS Credential Dumping)
2. Прочитай: саб-техники, Detection, Mitigations, какие группы используют
3. Открой ATT&CK Navigator → создай layer → подсвети техники, которые ты
   уже понимаешь по детекту (из разделов 01/04/09)
```
Задание: для 5 атак из своих разделов найди их Technique ID на сайте MITRE и запиши тактику.

💬 **Промпт мне:** «Объясни разницу Tactic/Technique/Sub-technique/Procedure на конкретном примере (например T1059.001). И покажи, как читать страницу техники на attack.mitre.org».

### Тактики Enterprise
- [ ] TA0043 Reconnaissance, TA0042 Resource Development, TA0001 Initial Access, TA0002 Execution, TA0003 Persistence, TA0004 Privilege Escalation, TA0005 Defense Evasion, TA0006 Credential Access, TA0007 Discovery, TA0008 Lateral Movement, TA0009 Collection, TA0011 C2, TA0010 Exfiltration, TA0040 Impact

💬 **Промпт мне:** «Прогони меня по 14 тактикам Enterprise по порядку: ты называешь тактику, я говорю что это и пример техники, ты проверяешь».

---

## Связки техника → детект (тренировать)

- [ ] T1003.001 LSASS dump → Sysmon EID 10
- [ ] T1053.005 Scheduled Task → Event 4698
- [ ] T1059.001 PowerShell → Event 4104
- [ ] T1547.001 Run keys → Sysmon EID 13
- [ ] T1021.001 RDP → Event 4624 type 10
- [ ] T1486 Ransomware → массовое изменение файлов

🛠 **Потрогать:** Возьми Sigma-репозиторий ([[10 - СЗИ и детект]]) и найди правила с тегами этих техник — увидишь реальную детект-логику.

💬 **Промпт мне:** «Сделай мне колоду из 20 карточек техника↔детект: с одной стороны Technique ID, с другой — артефакт/EventID. Прогоняй меня по ним вразнобой».

---

## Сопутствующее

- [ ] MITRE D3FEND (контрмеры)
- [ ] Detection engineering: Sigma-правила, маппинг на ATT&CK
- [ ] ATT&CK Navigator — карта покрытия

---

## Progress

- [ ] Раздел MITRE/Kill Chain пройден полностью
- [ ] Разложил свои атаки по тактикам на сайте MITRE
- [ ] Поработал в ATT&CK Navigator

---

## 🔭 Связь со следующими шагами

- **→ Blueteam (твоя цель):** ATT&CK — язык всей профессии. Detection engineering, threat hunting, отчёты TI — всё мапится на матрицу. Дальше: писать Sigma по технике, строить карту покрытия в Navigator.
- **→ Все направления:** ATT&CK — общий язык между red и blue. Пентестер описывает находки техниками, аналитик детектит техники. Знание матрицы обязательно везде.

---

## 🧪 Тест для повторения

> [!question]- 1. Перечисли 7 этапов Cyber Kill Chain.
> Reconnaissance → Weaponization → Delivery → Exploitation → Installation → Command & Control → Actions on Objectives. Цель защиты — разорвать цепочку как можно раньше.

> [!question]- 2. Разница Tactic / Technique / Procedure.
> Tactic — «зачем» (цель этапа, напр. Persistence TA0003). Technique — «как» (способ, напр. T1053 Scheduled Task). Procedure — конкретная реализация техники актором/инструментом.

> [!question]- 3. К какой тактике относится OS Credential Dumping и её ID?
> Credential Access (TA0006), техника T1003. Саб: LSASS Memory — T1003.001, DCSync — T1003.006.

> [!question]- 4. Сопоставь T1059.001 с детектом.
> T1059.001 — PowerShell (тактика Execution). Детект: Event 4104 (ScriptBlock), 4103 (module), Sysmon EID 1 на powershell.exe с подозрительными аргументами.

> [!question]- 5. Зачем ATT&CK Navigator?
> Визуализация покрытия детектами (тепловая карта матрицы): видно, какие техники закрыты правилами, а какие — слепые зоны. Приоритизация разработки детектов.

> [!question]- 6. Что такое Sigma и связь с ATT&CK?
> Открытый формат правил детекта, конвертируется под SIEM. Правила тегируются Technique ID → маппинг детектов на матрицу, измерение покрытия.

> [!question]- 7. Назови 5 тактик Enterprise по порядку атаки.
> Initial Access → Execution → Persistence → Privilege Escalation → Defense Evasion → Credential Access → Discovery → Lateral Movement → Collection → C2 → Exfiltration → Impact.

> [!question]- 8. Чем Kill Chain отличается от ATT&CK?
> Kill Chain — линейная модель из 7 этапов (высокоуровневая, последовательная). ATT&CK — детальная матрица из тактик и сотен конкретных техник, не строго линейная (техники переиспользуются на разных этапах).

> [!question]- 9. Что такое MITRE D3FEND?
> База контрмер (defensive countermeasures), дополняющая ATT&CK: для атакующих техник описывает защитные меры. Помогает связать «что детектим» с «как защищаемся».

> [!question]- 10. Что такое Resource Development в ATT&CK?
> TA0042 — этап, где атакующий готовит ресурсы до атаки: покупает домены/серверы, создаёт малварь, аккаунты, инфраструктуру C2. Предшествует Initial Access.

> [!question]- 11. Что такое процедура (procedure) на примере?
> Конкретная реализация техники. Напр. техника T1003.001 (LSASS dump), а процедура — «группа APT29 использует comsvcs.dll MiniDump». Один и тот же приём, разные конкретные исполнения.

> [!question]- 12. Как SOC использует ATT&CK в ежедневной работе?
> Размечает алерты техниками, строит отчёты/дашборды по покрытию, приоритизирует разработку детектов по слепым зонам, описывает инциденты единым языком, мапит TI-отчёты на свою инфраструктуру.

---

## 📚 Все источники раздела (сводно)

- Энциклопедия Касперского — MITRE ATT&CK: https://encyclopedia.kaspersky.ru/glossary/mitre-attack/
- Blue Team Cookbook — MITRE для SOC: https://vasilisa-l.gitbook.io/blue-team-cookbook/soc/mitre-attack
- Habr — обзор тактик разведки: https://habr.com/ru/articles/954656/
- Habr (Panda) — Cyber Kill Chain: https://habr.com/ru/company/panda/blog/327488/
- Habr (SecurityVision) — выстраивание килчейна: https://habr.com/ru/companies/securityvison/articles/777790/
- MITRE ATT&CK: https://attack.mitre.org/
- ATT&CK Navigator: https://mitre-attack.github.io/attack-navigator/
- Sigma: https://github.com/SigmaHQ/sigma
- Mindmap (друзья): https://xmind.app/m/WwtB/#
