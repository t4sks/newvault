---
tags: [soc, linux, roadmap]
section: 01
---

# 01 - Linux

[[00 - SOC Junior Roadmap (BiZone)]]

> **Лаба для этого раздела.** Заведи в VirtualBox/VMware две ВМ: **Ubuntu Server 22.04** (основная, на ней всё трогаем) и опционально **Kali Linux** (атакующий, понадобится позже). Сделай snapshot чистой Ubuntu сразу после установки — будешь откатываться после «грязных» экспериментов с persistence. Все команды ниже выполняются на Ubuntu, если не сказано иное.
>
> **Формат каждого пункта:** 📖 что прочитать · 🛠 что сделать руками в ВМ · 💬 какой промпт написать мне (Claude), если застрял.

---

## 🗺 Карта раздела: что атакующий делает в Linux и где ты это ловишь

Держи эту логику в голове — она связывает все пункты ниже в единую историю атаки (и привязана к тактикам MITRE из [[08 - MITRE ATT&CK и Kill Chain]]):

| Этап атаки | Что делает злоумышленник | Где ты (SOC) это видишь |
|---|---|---|
| Initial Access | RCE через веб / подбор SSH | `auth.log`, логи веб-сервера |
| Execution | запускает команды, скрипты | `.bash_history`, auditd execve, процессы |
| Persistence | cron, systemd, authorized_keys, ld.so.preload | изменённые файлы, новые юниты, auditd-watch |
| Priv Esc | SUID, sudo-misconfig, ядро | `auth.log` (sudo), нестандартный SUID |
| Defense Evasion | чистит логи, timestomp | пропуски в логах, расхождение времени |
| C2 / Exfil | туннели, исходящие соединения | `ss`, netflow, аномальный трафик |

> **Главный принцип SOC:** на каждую строку «что сделал атакующий» ты должен уметь ответить «каким артефактом/логом я это обнаружу». Все 🛠-задания ниже тренируют именно это.

## 🎯 Сквозной кейс раздела (собери в конце)

Когда пройдёшь все пункты, выполни **финальную лабу-историю** на snapshot-ВМ: ты — атакующий, потом ты — аналитик.
1. Подключись по SSH, добавь свой ключ в `authorized_keys` (Persistence).
2. Поставь cron на каждую минуту + systemd timer (Persistence x2).
3. Сделай `find` SUID-рутовым и поднимись до root (Priv Esc).
4. Дозапиши строку в `.bashrc` (Persistence).
5. Очисти `.bash_history` (Defense Evasion).
6. **Смени роль на аналитика:** запусти свой `hunt.sh` + проверь auditd + `last`/`lastb` и найди ВСЁ, что ты сделал. Запиши отчёт: какой артефакт выдал каждое действие.

Это и есть мини-версия реальной работы. Промпт мне: «Проверь мой отчёт по сквозному кейсу Linux, укажи, какие артефакты я пропустил».

---

## Базовое окружение

### Структура файловой системы (FHS)
- [ ] `/etc`, `/var`, `/tmp`, `/home`, `/usr`, `/opt`, `/proc`, `/dev` — что где живёт

📖 **Читать:** ArchWiki «File Hierarchy» (рус, самое чёткое) https://wiki.archlinux.org/title/File_hierarchy ; стандарт FHS кратко на losst https://losst.pro/struktura-katalogov-linux

🛠 **Потрогать:**
```bash
ls -la /                      # корень — посмотри все каталоги
cat /etc/os-release           # что за дистрибутив (/etc = конфиги)
ls /var/log                   # /var = изменяемые данные (логи!)
ls -la /tmp                   # временные файлы, sticky bit (см. ниже)
cat /proc/cpuinfo             # /proc = виртуальная ФС ядра, не файлы на диске
ls /proc/1/                   # каталог процесса с PID 1 (systemd)
```
Задание: открой `/proc/self/status` и найди строку с PID — это «ты сам». Объясни себе, почему `/proc` не занимает место на диске.

💬 **Промпт мне:** «Объясни, зачем SOC-аналитику знать FHS — приведи 3 примера, где злоумышленник прячет файлы в нестандартных местах (/tmp, /dev/shm, /var/tmp) и как это искать командой find».

### Права доступа
- [ ] rwx, чтение вывода `ls -l`, `chmod`, `chown`, числовая нотация (755, 644, 600)

📖 **Читать:** Habr «Права доступа в Linux» https://habr.com/ru/articles/667300/ ; интерактивный тренажёр chmod https://chmod-calculator.com/

🛠 **Потрогать:**
```bash
touch testfile && ls -l testfile      # разбери вывод: тип, u/g/o, владелец
chmod 600 testfile && ls -l testfile  # rw------- только владелец
chmod 755 testfile && ls -l testfile  # rwxr-xr-x
sudo chown root:root testfile && ls -l testfile
echo $?                               # проверь, можешь ли теперь писать в файл
```
Задание: создай файл, выставь права так, чтобы группа могла читать, но не писать; владелец — всё; остальные — ничего. Запиши, какое это число.

💬 **Промпт мне:** «Дай мне 5 задачек на перевод rwx в восьмеричную нотацию и обратно, проверь мои ответы».

### SUID / SGID / sticky bit
- [ ] что это, чем опасны, как искать (`find / -perm -4000`)

📖 **Читать:** Xakep «Эскалация привилегий через SUID» https://xakep.ru/2018/05/07/suid-sgid-privesc/ ; GTFOBins https://gtfobins.github.io/ (поиск конкретных бинарей)

🛠 **Потрогать:**
```bash
find / -perm -4000 -type f 2>/dev/null   # все SUID-бинари в системе
ls -l /usr/bin/passwd                    # классический SUID (буква s вместо x)
ls -l /usr/bin/sudo                      # ещё пример
find / -perm -2000 -type f 2>/dev/null   # SGID
ls -ld /tmp                              # sticky bit (буква t в конце)
```
Задание (privesc-демо, безопасно в ВМ): сделай `find` SUID-рутовым — `sudo chmod u+s /usr/bin/find` — затем выполни от обычного юзера `find . -exec /bin/sh -p \; -quit` и посмотри `id`. Получил root? Откати snapshot. Это твоя первая эскалация — пригодится в пентесте.

💬 **Промпт мне:** «Я нашёл нестандартный SUID-бинарь /usr/local/bin/X в выводе find. Как мне понять, опасен ли он? Дай чеклист проверки и команды».

### Пользователи и группы
- [ ] `/etc/passwd`, `/etc/shadow`, `/etc/group` — формат и что там лежит

📖 **Читать:** losst «/etc/passwd и /etc/shadow» https://losst.pro/fajl-etc-passwd-v-linux ; man 5 passwd

🛠 **Потрогать:**
```bash
cat /etc/passwd                # разбери поля: имя:x:UID:GID:comment:home:shell
sudo cat /etc/shadow           # хеши паролей (формат $6$ = SHA512)
cat /etc/group
sudo useradd -m hacker -s /bin/bash && sudo passwd hacker
grep hacker /etc/passwd /etc/shadow
grep -E ':0:' /etc/passwd      # ищем всех с UID 0 (должен быть только root!)
```
Задание: создай юзера, дай ему UID 0 вручную в /etc/passwd, объясни почему это бэкдор и как SOC такое детектит.

💬 **Промпт мне:** «Покажи, как по /etc/passwd и /etc/shadow найти признаки компрометации: лишние root-аккаунты, юзеры с пустым паролем, нестандартные shell. Дай готовые grep-команды».

### Процессы
- [ ] `ps`, `top`, `/proc/<pid>`, родитель-потомок, дерево `pstree`

📖 **Читать:** Habr «Процессы в Linux» https://habr.com/ru/articles/352338/ ; man ps

🛠 **Потрогать:**
```bash
ps aux                         # все процессы
ps auxf                        # дерево (родитель-потомок)
pstree -p                      # наглядное дерево с PID
top                            # живой мониторинг (q для выхода)
ls -l /proc/$$/                # каталог твоей текущей оболочки
cat /proc/1/cmdline | tr '\0' ' '   # чем запущен PID 1
ls -l /proc/<PID>/exe          # путь к исполняемому файлу процесса
ls -l /proc/<PID>/cwd          # рабочий каталог процесса
```
Задание: запусти `sleep 600 &`, найди его PID, посмотри его `/proc/<PID>/`, определи родителя через `ps -o ppid= -p <PID>`. Это база форензики — связать процесс с родителем.

💬 **Промпт мне:** «Дай мне сценарий: подозрительный процесс с именем kworker слушает порт. Какие команды (ps, ls /proc, ss, lsof) и в каком порядке использовать, чтобы понять — это легит ядро или малварь под него маскируется?»

---

## Bash и оболочка

### Основы оболочки
- [ ] Что такое bash, отличие от sh/zsh; конвейер `|`; `&&`, `||`, `;`; `$(...)`; переменные окружения

📖 **Читать:** «Bash-скрипты с нуля» FirstVDS https://firstvds.ru/technology/bash-skript ; tldr.sh для команд

🛠 **Потрогать:**
```bash
echo $SHELL                          # какая оболочка
cat /etc/shells                      # доступные оболочки
ls /etc | grep conf && echo "есть"   # конвейер + &&
false || echo "сработало по ИЛИ"
echo "сегодня $(date)"               # подстановка команды
export MYVAR=test && echo $MYVAR
env | grep PATH                      # переменные окружения
```

💬 **Промпт мне:** «Объясни разницу между sh, bash и zsh с точки зрения форензики: почему важно знать, какой shell у юзера в /etc/passwd, и где смотреть историю каждого».

### Перенаправление потоков (stdin/stdout/stderr)
- [ ] Потоки 0/1/2; `command 2>&1`; отличие `2>&1 > file` от `> file 2>&1` (порядок важен)

📖 **Читать:** Selectel https://selectel.ru/blog/tutorials/linux-redirection/ ; разбор порядка на k-max https://www.k-max.name/linux/programmnye-kanaly-i-potoki-perenapravlenie/

🛠 **Потрогать:**
```bash
ls /etc /nonexistent              # увидишь stdout И stderr вперемешку
ls /etc /nonexistent > out.txt    # в файл только stdout, ошибка на экране
ls /etc /nonexistent 2> err.txt   # в файл только stderr
ls /etc /nonexistent > all.txt 2>&1   # оба в файл
ls /etc /nonexistent 2>&1 > x.txt     # сравни: ошибка осталась на экране!
cat out.txt err.txt
```
Задание: воспроизведи оба варианта (`2>&1 > file` и `> file 2>&1`), объясни вслух разницу. Это вопрос с собеседования.

![[image (1) 1.png]]


💬 **Промпт мне:** «Проверь моё понимание: я думаю что [твоё объяснение]. Правильно? Дай 3 каверзных примера на перенаправление с ответами под спойлер».

### Обработка текста и логов
- [ ] `grep`, `awk`, `sed`, `cut`, `sort`, `uniq`, `wc` — на лету парсить логи (главный навык SOC!)

📖 **Читать:** «grep/awk/sed для анализа логов» https://habr.com/ru/articles/582450/ ; explainshell.com (разбирает любую команду)

🛠 **Потрогать (на реальном логе):**
```bash
# топ IP по числу обращений в auth.log
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head
sudo grep sshd /var/log/auth.log | cut -d' ' -f1-3,9-   # время + событие
sudo journalctl -u ssh --no-pager | grep -i fail
```
Задание: сгенерируй неудачные входы (`ssh wrong@localhost` несколько раз с неверным паролем), потом одной командой выведи топ-5 атакующих IP/юзеров.

💬 **Промпт мне:** «Дай 10 однострочников на grep/awk/sed для типовых задач SOC-аналитика: топ IP, выборка по времени, подсчёт событий, фильтр по нескольким условиям. С пояснением каждой».

---

## Запланированные задачи

### cron, at, anacron, systemd timers
- [ ] cron: `/etc/crontab`, `/etc/cron.d/`, `/var/spool/cron/crontabs/`, `crontab -l`
- [ ] `at`, anacron
- [ ] systemd timers: `systemctl list-timers`, `.timer` + `.service`
- [ ] Детект: подозрительный cron / скрытые таймеры

📖 **Читать:** Habr persistence (раздел cron) https://habr.com/ru/articles/568298/ ; ArchWiki systemd/Timers https://wiki.archlinux.org/title/Systemd/Timers

🛠 **Потрогать:**
```bash
crontab -l                                  # cron текущего юзера
sudo cat /etc/crontab                       # системный
ls -la /etc/cron.d/ /etc/cron.daily/
sudo ls -la /var/spool/cron/crontabs/       # тут юзерские
systemctl list-timers --all                 # systemd-таймеры
# создай свой cron на каждую минуту:
(crontab -l 2>/dev/null; echo "* * * * * echo hi >> /tmp/cronlog") | crontab -
sleep 70 && cat /tmp/cronlog                # проверь что отработал
```
Задание: создай systemd timer + service, которые пишут в файл раз в минуту. Потом найди их как «аналитик» через `systemctl list-timers` и в `/etc/systemd/`.

💬 **Промпт мне:** «Помоги составить .timer и .service юнит, который раз в минуту дописывает дату в /tmp/beacon.log. Дай содержимое обоих файлов и команды enable/start».

---

## Методы закрепления (persistence) — атака + детект

> Ключевой блок для SOC. Каждый механизм учим связкой: **поставил закреп → нашёл его как аналитик**. Делай на snapshot-ВМ, откатывай после.

- [ ] cron jobs → детект: аудит изменений crontab-файлов
- [ ] systemd services/timers → детект: новые юниты в `/etc/systemd/`
- [ ] `.bashrc`, `.bash_profile`, `/etc/profile`, `/etc/profile.d/` → детект: модификация rc-файлов
- [ ] SSH `authorized_keys` → детект: новые ключи
- [ ] `init.d` / `rc.local`
- [ ] `LD_PRELOAD` / `/etc/ld.so.preload` (хукинг библиотек)
- [ ] вредоносные PAM-модули
- [ ] web shell, malicious kernel module (rootkit)
- [ ] MOTD, udev rules

📖 **Читать:** Habr «Закрепление в Linux» (основная, проходит почти все эти техники) https://habr.com/ru/articles/568298/ ; MITRE ATT&CK Persistence для Linux https://attack.mitre.org/tactics/TA0003/

🛠 **Потрогать (поставь закреп → задетекть):**
```bash
# 1. SSH-ключ как закреп
ssh-keygen -t ed25519 -f /tmp/k -N ""
cat /tmp/k.pub >> ~/.ssh/authorized_keys
# детект: когда менялся файл?
ls -la --time-style=full-iso ~/.ssh/authorized_keys

# 2. rc-файл
echo 'echo "backdoor loaded"' >> ~/.bashrc
# детект:
grep -rn "echo" ~/.bashrc /etc/profile.d/ 2>/dev/null

# 3. LD_PRELOAD (концепт)
echo "/tmp/evil.so" | sudo tee /etc/ld.so.preload   # НЕ создавай .so, просто посмотри
sudo cat /etc/ld.so.preload
sudo rm /etc/ld.so.preload   # убери сразу
```
Задание: поставь 3 разных закрепа, потом напиши свой bash-скрипт `hunt.sh`, который их все находит (cron + authorized_keys + ld.so.preload + изменённый .bashrc). Откати snapshot.

**Сводная таблица «закреп → где живёт → как детектить»** (выучи наизусть — спрашивают):

| Механизм | Где живёт | Артефакт для детекта |
|---|---|---|
| cron | `/etc/cron*`, `/var/spool/cron/crontabs/` | изменение mtime файлов, auditd-watch |
| systemd | `/etc/systemd/system/`, `~/.config/systemd/user/` | новые `.service`/`.timer`, `systemctl list-timers` |
| SSH key | `~/.ssh/authorized_keys` | новая строка, mtime, auditd-watch |
| rc-файлы | `.bashrc`, `/etc/profile.d/`, `/etc/profile` | новые строки, mtime |
| LD_PRELOAD | `/etc/ld.so.preload`, env `LD_PRELOAD` | существование файла (норм. его НЕТ) |
| PAM | `/etc/pam.d/`, `/lib/.../security/` | изменение модулей, чужие `.so` |
| rc.local/init.d | `/etc/rc.local`, `/etc/init.d/` | новые строки/скрипты |
| kernel module | `lsmod`, `/etc/modules` | скрытые модули, несоответствие lsmod и /proc/modules |
| udev | `/etc/udev/rules.d/` | правила с RUN+= на исполнение |
| MOTD | `/etc/update-motd.d/` | исполняемые скрипты в каталоге |

**Дополнительно потрогать (PAM и kernel — продвинутые):**
```bash
# PAM: посмотри модули аутентификации (вредоносный PAM перехватывает пароли)
ls -la /etc/pam.d/
cat /etc/pam.d/sshd
ls -la /lib/x86_64-linux-gnu/security/ 2>/dev/null || ls /lib/security/ 2>/dev/null

# Kernel modules (руткиты прячутся тут):
lsmod                                # загруженные модули
cat /proc/modules | head
# детект скрытого модуля: сравни вывод lsmod и /sys/module/
ls /sys/module/ | sort > /tmp/a; lsmod | awk 'NR>1{print $1}' | sort > /tmp/b; diff /tmp/a /tmp/b

# Web shell (если поднят веб-сервер): ищи свежие исполняемые скрипты в webroot
find /var/www -name "*.php" -mtime -7 2>/dev/null   # php-файлы за неделю
```
Задание: посмотри `/etc/pam.d/sshd`, объясни, как вредоносный PAM-модуль мог бы логировать пароли. Проверь `lsmod` на наличие неизвестных модулей.

💬 **Промпт мне:** «Помоги написать скрипт hunt.sh для baseline-проверки персистентности на Linux: cron, systemd, authorized_keys, ld.so.preload, rc-файлы, PAM, kernel modules, udev, MOTD. Каждую находку — с пояснением, почему подозрительно, и с привязкой к технике MITRE».

---

## Логи Linux

- [ ] `/var/log/` — что где лежит
- [ ] `/var/log/auth.log` (Debian) / `/var/log/secure` (RHEL) — аутентификация, sudo, ssh
- [ ] `/var/log/syslog`, `/var/log/messages`
- [ ] `journalctl` (systemd journal)
- [ ] `wtmp`, `btmp`, `utmp` — логины (`last`, `lastb`, `who`)
- [ ] `.bash_history`
- [ ] auditd: правила, `ausearch`, `aureport`
- [ ] Детект очистки логов (антифорензика)

📖 **Читать:** Habr (Angara) «Топ-10 артефактов Linux» https://habr.com/ru/companies/angarasecurity/articles/767124/ ; auditd — Xakep https://xakep.ru/2021/09/16/linux-audit/ ; расшифровка audit.log https://itmag.pro/unix/common/linux-auditlog-explain

🛠 **Потрогать:**
```bash
sudo tail -f /var/log/auth.log    # в другом окне делай ssh — смотри живьём
last                              # успешные входы (wtmp)
sudo lastb                        # неудачные (btmp)
who && w                          # кто сейчас в системе
cat ~/.bash_history               # история команд
journalctl -p err -b              # ошибки текущей загрузки
journalctl _COMM=sshd --no-pager  # всё от sshd

# auditd
sudo apt install auditd -y
sudo auditctl -w /etc/passwd -p wa -k passwd_watch   # следить за /etc/passwd
sudo useradd testaudit                                # триггерим
sudo ausearch -k passwd_watch                         # ищем событие
sudo aureport --summary
```
Задание: настрой auditd-правило на запись в `~/.ssh/authorized_keys`, добавь ключ, найди событие через `ausearch`. Потом «замети следы» (`echo > ~/.bash_history`) и подумай, как аналитик это обнаружит.

💬 **Промпт мне:** «Дай набор базовых auditd-правил для SOC: отслеживание /etc/passwd, /etc/shadow, authorized_keys, ld.so.preload, исполнения из /tmp. Объясни синтаксис каждого правила (-w, -p, -k)».

---

## SELinux / AppArmor

- [ ] MAC (Mandatory Access Control) vs DAC
- [ ] SELinux: режимы (enforcing/permissive/disabled), контексты, типы
- [ ] AppArmor: профили (по умолчанию в Ubuntu!)
- [ ] Зачем для SOC: ограничение скомпрометированных процессов

📖 **Читать:** Habr «SELinux: быстрый онбординг» https://habr.com/ru/articles/1035908/ ; MAC vs DAC https://habr.com/ru/companies/cloud4y/articles/920600/ ; AppArmor в Ubuntu https://wiki.archlinux.org/title/AppArmor

🛠 **Потрогать (AppArmor — он уже в Ubuntu):**
```bash
sudo aa-status                    # статус AppArmor, какие профили активны
ls /etc/apparmor.d/               # профили
sudo apparmor_status
# SELinux (если будешь на CentOS/Rocky-ВМ):
# getenforce ; sestatus ; ls -Z /etc/passwd  (контекст файла)
```
Задание: посмотри профиль AppArmor для какого-нибудь демона, объясни, что он ограничивает и зачем это нужно при компрометации сервиса.

💬 **Промпт мне:** «Объясни на примере: веб-сервер nginx скомпрометирован через RCE. Как AppArmor/SELinux ограничивает, что атакующий сможет сделать дальше? Чем MAC спасает там, где DAC бессилен».

---

## Progress

- [ ] Раздел Linux пройден полностью
- [ ] Все 🛠 задания выполнены в ВМ
- [ ] Написал свой `hunt.sh` для детекта persistence

---

## ⚠️ Частые ошибки на собеседовании (Linux)

- **Путать `>` и `>>`** — первое перезаписывает, второе дописывает. На вопросе про перенаправление это базовая проверка.
- **Говорить «логи в /var/log/syslog» и всё** — назови конкретику: auth.log/secure для аутентификации, btmp для неудачных входов, journal для systemd.
- **Не знать, что `last` читает бинарный wtmp, а не текстовый лог** — это частый уточняющий вопрос.
- **Считать, что root всесилен при SELinux enforcing** — нет, MAC ограничивает даже root.
- **На вопрос про persistence называть только cron** — назови минимум 5 механизмов из таблицы выше.
- **Забывать про детект-сторону** — если назвал атаку, сразу говори, каким артефактом её ловишь. Это то, что отличает SOC-кандидата от просто «знающего Linux».
- **Путать SUID и sudo** — SUID это бит на файле (запуск с правами владельца), sudo это утилита повышения прав по политике `/etc/sudoers`.

---

## 🔭 Связь со следующими шагами (AppSec / инфрапентест / blueteam)

- **→ Инфрапентест:** SUID-эскалация и LD_PRELOAD здесь — это первые шаги Linux privilege escalation. Дальше: курс «Linux PrivEsc» на TryHackMe, LinPEAS, GTFOBins. После SOC возьмёшь это вглубь.
- **→ AppSec:** понимание прав, процессов и того, как RCE в приложении превращается в доступ к ОS — основа. Web shell из блока persistence напрямую связан с [[02 - Web (OWASP)]].
- **→ Blueteam (твоя цель):** auditd + детект persistence + парсинг логов grep/awk — это ядро Linux-форензики. Дальше: Velociraptor, Sigma-правила для Linux, Sysmon for Linux. Эта заметка — фундамент под всё это.

---

## 🧪 Тест для повторения (ответы скрыты — клик разворачивает)

> [!question]- 1. Чем `2>&1 > file` отличается от `> file 2>&1`?
> `> file 2>&1` — stdout уходит в файл, затем stderr направляется туда же → оба в файле. `2>&1 > file` — stderr сначала дублирует текущий stdout (терминал), потом stdout перенаправляется в файл → stderr остаётся в терминале, stdout в файле. Порядок интерпретируется слева направо.

> [!question]- 2. Где искать запланированные задачи в Linux (минимум 4 места)?
> `/etc/crontab`, `/etc/cron.d/`, `/etc/cron.{daily,hourly,weekly}/`, пользовательские `/var/spool/cron/crontabs/`, systemd timers (`systemctl list-timers`), `at`/atd, anacron.

> [!question]- 3. Назови 5 механизмов persistence в Linux и артефакт для детекта каждого.
> cron (изменение crontab-файлов), systemd service/timer (новые юниты в `/etc/systemd/`), `~/.ssh/authorized_keys` (новый ключ), rc-файлы `.bashrc`/`/etc/profile.d/` (модификация), `/etc/ld.so.preload` (LD_PRELOAD хук), вредоносный PAM-модуль, malicious kernel module.

> [!question]- 4. Где хранятся логи аутентификации и как посмотреть неудачные входы?
> Debian: `/var/log/auth.log`, RHEL: `/var/log/secure`. Неудачные логины — `lastb` (читает `/var/log/btmp`), успешные — `last` (`wtmp`). Через systemd — `journalctl _COMM=sshd`.

> [!question]- 5. Что такое SUID и почему опасен? Как найти все SUID-бинари?
> SUID — бит, при котором программа выполняется с правами владельца файла (часто root), а не запустившего. Опасен: уязвимый/нештатный SUID-бинарь = эскалация до root. Поиск: `find / -perm -4000 -type f 2>/dev/null`. См. GTFOBins.

> [!question]- 6. SELinux: режимы работы и чем MAC отличается от DAC?
> Режимы: enforcing (блокирует), permissive (логирует, не блокирует), disabled. DAC — права назначает владелец (rwx). MAC — мандатный контроль через политику системы поверх DAC; даже root ограничен контекстами/типами.

> [!question]- 7. Что такое конвейер (pipe) и приведи пример анализа логов.
> `|` передаёт stdout одной команды на stdin другой. Пример: `grep Failed auth.log | awk '{print $NF}' | sort | uniq -c | sort -rn` — топ источников неудачных входов.

> [!question]- 8. Разбери поля строки /etc/passwd.
> `name:x:UID:GID:comment:home:shell`. x = пароль вынесен в shadow. Признак бэкдора: UID=0 не у root, или shell у системной учётки (должен быть nologin/false).

> [!question]- 9. Чем `last`, `lastb`, `who` отличаются?
> `last` — история успешных входов (читает бинарный wtmp). `lastb` — неудачные входы (btmp). `who`/`w` — кто залогинен прямо сейчас (utmp).

> [!question]- 10. Как атакующий чистит следы и как это детектить?
> Очистка `.bash_history` (`echo >`, `unset HISTFILE`), truncate логов, timestomp. Детект: пропуски/обнуление в логах, расхождение времени файла, центральная отправка логов (syslog на отдельный сервер) — тогда локальная чистка бесполезна.

> [!question]- 11. Что делает auditd и чем лучше обычных логов?
> Подсистема аудита ядра: пишет события по правилам (доступ к файлам -w, системные вызовы -S) с привязкой к процессу/пользователю. Ловит то, что обычные логи не видят: кто читал /etc/shadow, кто менял authorized_keys, execve команд.

> [!question]- 12. Зачем атакующему /tmp и /dev/shm?
> Это writable-каталоги, часто монтируются без noexec в дефолте, удобны для дропа и запуска нагрузки. /dev/shm — в памяти (tmpfs), не оставляет следов на диске. Детект: исполнение из /tmp, /dev/shm через auditd.

> [!question]- 13. Что такое LD_PRELOAD-атака?
> Подмена/перехват функций библиотек: указанная в LD_PRELOAD (env) или /etc/ld.so.preload .so загружается раньше системных и переопределяет их (хук). Используется для руткитов, кражи паролей. Детект: наличие /etc/ld.so.preload (в норме его нет), подозрительный LD_PRELOAD в env процессов.

> [!question]- 14. Где systemd хранит юниты и как найти вредоносный таймер?
> Системные: `/etc/systemd/system/`, `/lib/systemd/system/`. Пользовательские: `~/.config/systemd/user/`. Поиск: `systemctl list-timers --all`, проверка свежих/нестандартных `.service` и `.timer`, на что указывает ExecStart.

> [!question]- 15. Как RCE в веб-приложении превращается в полный контроль хоста (цепочка)?
> Web shell (запись .php в webroot) → выполнение команд от www-data → разведка (id, sudo -l, SUID) → privilege escalation (SUID/sudo-misconfig/kernel) → root → persistence (cron/ssh key). Каждый шаг оставляет артефакт: новый файл, процесс, sudo в auth.log.

> [!question]- 16. Что показывает /proc/<pid>/ и зачем форензике?
> Виртуальные данные процесса: `exe` (путь к бинарю, даже если удалён с диска), `cwd` (рабочий каталог), `cmdline` (аргументы запуска), `environ` (переменные, в т.ч. LD_PRELOAD), `fd/` (открытые файлы/сокеты). Позволяет восстановить, что за процесс, даже если файл удалён.

---

## 📚 Все источники раздела (сводно)

**Bash / потоки**
- Selectel — перенаправление: https://selectel.ru/blog/tutorials/linux-redirection/
- k-max — порядок `2>&1`: https://www.k-max.name/linux/programmnye-kanaly-i-potoki-perenapravlenie/
- FirstVDS — bash с нуля: https://firstvds.ru/technology/bash-skript
- grep/awk/sed для логов: https://habr.com/ru/articles/582450/
- explainshell (разбор команд): https://explainshell.com/

**ФС / права / процессы**
- ArchWiki File hierarchy: https://wiki.archlinux.org/title/File_hierarchy
- Habr — права доступа: https://habr.com/ru/articles/667300/
- Xakep — SUID/SGID privesc: https://xakep.ru/2018/05/07/suid-sgid-privesc/
- GTFOBins: https://gtfobins.github.io/
- losst — /etc/passwd, /etc/shadow: https://losst.pro/fajl-etc-passwd-v-linux
- Habr — процессы: https://habr.com/ru/articles/352338/

**Persistence + детект**
- Habr — Закрепление в Linux (основная): https://habr.com/ru/articles/568298/
- MITRE Persistence: https://attack.mitre.org/tactics/TA0003/

**Логи / auditd**
- Habr (Angara) — Топ-10 артефактов Linux: https://habr.com/ru/companies/angarasecurity/articles/767124/
- Xakep — auditd: https://xakep.ru/2021/09/16/linux-audit/
- itmag — чтение audit.log: https://itmag.pro/unix/common/linux-auditlog-explain

**SELinux / AppArmor**
- Habr — SELinux онбординг: https://habr.com/ru/articles/1035908/
- Habr — MAC vs DAC: https://habr.com/ru/companies/cloud4y/articles/920600/

**Практика целиком**
- TryHackMe — Linux Fundamentals 1-3 (бесплатно): https://tryhackme.com/module/linux-fundamentals
- OverTheWire Bandit (игра на bash/Linux в SSH): https://overthewire.org/wargames/bandit/
