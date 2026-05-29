---
tags: [soc, c2, pivoting, dfir, roadmap]
section: 07
---

# 07 - Глубокие топики (C2, Pivoting, DFIR)

[[00 - SOC Junior Roadmap (BiZone)]]

## Pivoting (латеральное движение)

- [ ] Что такое pivoting — использование скомпрометированного хоста как точки входа в сегмент
- [ ] Методы: SSH-туннели, SOCKS-прокси, port forwarding
- [ ] Инструменты: chisel, ligolo-ng, proxychains, Metasploit autoroute, SSH `-D`/`-L`/`-R`
- [ ] Связь с [[03 - Сети и сетевые атаки]] (туннелирование) и DMZ
- [ ] Детект: нетипичные внутренние соединения, цепочки логонов между хостами

## C2-фреймворки (Command & Control)

- [ ] Концепция: beacon, listener, payload, callback, jitter
- [ ] Каналы связи: HTTP(S), DNS, SMB-named pipes, ICMP
- [ ] Фреймворки: Cobalt Strike, Sliver, Mythic, Metasploit/Meterpreter, Havoc, Empire
- [ ] Malleable C2 profiles (маскировка трафика под легитимный)
- [ ] Детект: beaconing-паттерны (регулярные интервалы), нетипичные User-Agent, JA3/JA4 fingerprint, DNS-аномалии

## Концепция DFIR (Digital Forensics & Incident Response)

- [ ] Этапы IR (NIST): Preparation → Detection & Analysis → Containment → Eradication → Recovery → Lessons Learned
- [ ] SANS PICERL: Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned
- [ ] Триаж артефактов: память, диск, сеть, логи
- [ ] Volatility (анализ памяти), KAPE, velociraptor
- [ ] Цепочка доказательств (chain of custody)
- [ ] Timeline analysis, super timeline (plaso)

## Forensics-артефакты (для детекта/расследования)

- [ ] Windows: Prefetch, Amcache, ShimCache, SRUM, Jump Lists, LNK, $MFT, ShellBags
- [ ] Linux: bash_history, auth.log, journal, временные метки inode
- [ ] Антифорензика: timestomping, очистка логов, secure delete

## Progress

- [ ] Раздел Глубокие топики пройден полностью

---

## 🧪 Тест для повторения

> [!question]- 1. Что такое pivoting и как детектить?
> Использование скомпрометированного хоста как трамплина в недоступный сегмент (SSH-туннель, SOCKS-прокси, port forwarding, proxychains). Детект: нетипичные внутренние соединения, цепочки логонов хост→хост, появление прокси-инструментов, аномальный трафик от рабочей станции вглубь сети.

> [!question]- 2. Что такое beaconing и как его ловят?
> Регулярные callback-обращения импланта к C2-серверу. Детект: периодичность интервалов (даже с jitter), повторяющиеся обращения к одному домену/IP, нетипичный User-Agent, JA3/JA4-фингерпринт TLS, малый объём но высокая регулярность.

> [!question]- 3. Назови 3 C2-фреймворка и каналы их связи.
> Cobalt Strike, Sliver, Mythic, Metasploit/Meterpreter, Havoc, Empire. Каналы: HTTP(S), DNS, SMB named pipes, ICMP. Malleable-профили маскируют трафик под легитимный.

> [!question]- 4. Этапы реагирования на инцидент (NIST/SANS).
> SANS PICERL: Preparation → Identification → Containment → Eradication → Recovery → Lessons Learned. NIST аналогично: Preparation → Detection & Analysis → Containment/Eradication/Recovery → Post-Incident.

> [!question]- 5. Forensics-артефакты Windows для определения «что запускалось».
> Prefetch (запуск программ), Amcache/ShimCache (исполнявшиеся бинари), SRUM (сетевая/процессная активность), Jump Lists и LNK (открытые файлы), $MFT (файловая активность), ShellBags (навигация по папкам).

> [!question]- 6. Что такое timestomping и как заметить?
> Антифорензика — подмена временных меток файла, чтобы скрыть момент создания. Заметить: расхождение $STANDARD_INFORMATION и $FILE_NAME времён в MFT, аномальные/round-секунды метки, несоответствие времени соседним файлам.

---

## 📚 Источники для подготовки

**Pivoting / латеральное перемещение**
- Habr — Pivoting или проброс портов (SSH-туннели, способы): https://habr.com/ru/articles/302168/
- Xakep — Большой проброс: оттачиваем pivoting на Hack The Box (шпаргалка): https://xakep.ru/2019/12/26/htb-pivoting/
- HackWare — Сетевой pivoting: понятие, примеры, техники, инструменты: https://hackware.ru/?p=9016
- Habr — PhantomCore: pivoting через легитимные инструменты + детект (EID 4103, WinRM/RDP): https://habr.com/ru/articles/1025674/

**C2-фреймворки / beaconing + детект (Cobalt Strike)**
- Red Canary — Cobalt Strike (детект-аналитика, named pipes, поведение): https://redcanary.com/threat-detection-report/threats/cobalt-strike/
- Unit42 (Palo Alto) — Cobalt Strike Malleable C2 (почему сложно детектить): https://unit42.paloaltonetworks.com/cobalt-strike-malleable-c2-profile/
- Elastic — Cobalt Strike C2 Beacon (готовое правило детекта + реагирование): https://www.elastic.co/guide/en/security/current/cobalt-strike-command-and-control-beacon.html

**DFIR (реагирование на инциденты)**
- SecurityLab — Основные принципы DFIR (этапы расследования): https://www.securitylab.ru/analytics/541880.php
- Инфобезопасность — DFIR: методы реагирования (PICERL): https://infobezopasnost.ru/blog/articles/dfir-metody-reagirovaniya-na-intsidenty/
- Habr — DFIR на практике, ч.1: Lockdown Lab (разбор атаки по MITRE, артефакты): https://habr.com/ru/articles/986314/
- Habr — Обзор NIST 800-61 по реагированию на инциденты: https://habr.com/ru/articles/904252/

**Forensics-артефакты**
- SANS DFIR cheat sheets: https://www.sans.org/posters/?focus-area=digital-forensics
