---
tags: [soc, linux, roadmap]
section: 01
---

# 01 - Linux

[[00 - SOC Junior Roadmap (BiZone)]]

## Базовое окружение

- [ ] Структура файловой системы (FHS): `/etc`, `/var`, `/tmp`, `/home`, `/usr`, `/opt`, `/proc`, `/dev`
- [ ] Права доступа: rwx, чтение `ls -l`, `chmod`, `chown`, числовая нотация (755, 644)
- [ ] SUID / SGID / sticky bit — что это, чем опасны (`find / -perm -4000`)
- [ ] Пользователи и группы: `/etc/passwd`, `/etc/shadow`, `/etc/group`
- [ ] Процессы: `ps`, `top`, `/proc/<pid>`, родитель-потомок, дерево процессов `pstree`

## Bash и оболочка

- [ ] Что такое bash, отличие от sh/zsh
- [ ] Конвейер команд (pipe `|`) — передача stdout одной команды на stdin другой
- [ ] Перенаправление потоков: stdin (0), stdout (1), stderr (2)
- [ ] Отладочная информация в stdout: `command 2>&1` (перенаправить stderr в stdout)
- [ ] Отличие `2>&1 > file` от `> file 2>&1` (порядок важен)
- [ ] `&&`, `||`, `;`, подстановка `$(...)`, переменные окружения
- [ ] `grep`, `awk`, `sed`, `cut`, `sort`, `uniq` — обработка логов

## Запланированные задачи

- [ ] cron: `/etc/crontab`, `/etc/cron.d/`, `/var/spool/cron/crontabs/`, `crontab -l`
- [ ] anacron, at
- [ ] systemd timers: `systemctl list-timers`, `.timer` + `.service` юниты
- [ ] Детект: подозрительный cron, скрытые таймеры

## Методы закрепления (persistence) — атака + детект

- [ ] cron jobs (см. выше) → детект: аудит изменений crontab-файлов
- [ ] systemd services / timers → детект: новые/изменённые юниты в `/etc/systemd/`
- [ ] `.bashrc`, `.bash_profile`, `/etc/profile`, `/etc/profile.d/` → детект: модификация rc-файлов
- [ ] SSH authorized_keys → детект: новые ключи в `~/.ssh/authorized_keys`
- [ ] init.d / rc.local
- [ ] LD_PRELOAD / `/etc/ld.so.preload` (хукинг библиотек)
- [ ] PAM-модули (вредоносные)
- [ ] Web shell, malicious kernel module (rootkit)
- [ ] MOTD, udev rules

## Логи Linux

- [ ] `/var/log/` — что где лежит
- [ ] `/var/log/auth.log` (Debian) / `/var/log/secure` (RHEL) — аутентификация, sudo, ssh
- [ ] `/var/log/syslog`, `/var/log/messages`
- [ ] `journalctl` (systemd journal)
- [ ] `wtmp`, `btmp`, `utmp` — логины (`last`, `lastb`, `who`)
- [ ] `.bash_history`
- [ ] auditd: правила, `ausearch`, `aureport`
- [ ] Детект очистки логов (антифорензика)

## SELinux / AppArmor

- [ ] Что такое MAC (Mandatory Access Control) vs DAC
- [ ] SELinux: режимы (enforcing/permissive/disabled), контексты, типы
- [ ] AppArmor: профили
- [ ] Зачем для SOC: ограничение compromised-процессов

## Progress

- [ ] Раздел Linux пройден полностью

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

---

## 📚 Источники для подготовки

**Bash / потоки / перенаправление (`2>&1`)**
- Selectel — Перенаправление ввода/вывода в Linux: https://selectel.ru/blog/tutorials/linux-redirection/
- FirstVDS — Bash-скрипты: перенаправление ввода и вывода: https://firstvds.ru/technology/perenapravleniya-vvoda-vyvoda-linux
- k-max.name — почему порядок `2>&1 > file` важен (с разбором): https://www.k-max.name/linux/programmnye-kanaly-i-potoki-perenapravlenie/
- OpenNet — дескрипторы 0/1/2, базовый справочник: https://www.opennet.ru/docs/RUS/bash_scripting_guide/c11620.html

**Persistence (закрепление) + детект**
- Habr — Закрепление в Linux. Linux Persistence (основная, читать первой): https://habr.com/ru/articles/568298/

**Логи и аудит**
- Habr (Angara) — Топ-10 артефактов Linux для расследования инцидентов: https://habr.com/ru/companies/angarasecurity/articles/767124/
- Habr (PT) — Учимся понимать события подсистемы аудита Linux: https://habr.com/ru/companies/pt/articles/789014/
- Habr — Настройка auditd для обнаружения и расследования инцидентов: https://habr.com/ru/articles/553036/
- Xakep — Основы аудита: журналирование важных событий в Linux: https://xakep.ru/2021/09/16/linux-audit/
- itmag.pro — Как читать audit.log (расшифровка полей): https://itmag.pro/unix/common/linux-auditlog-explain

**SELinux / MAC**
- Habr — SELinux: быстрый онбординг (типы, домены, политики, практика): https://habr.com/ru/articles/1035908/
- Habr — SELinux: описание и особенности, часть 1: https://habr.com/ru/companies/kingservers/articles/209644/
- Habr — Защита Debian через SELinux (MAC vs DAC простым языком): https://habr.com/ru/companies/cloud4y/articles/920600/

**Privilege escalation / SUID (для контекста)**
- GTFOBins (поиск abuse-векторов бинарей): https://gtfobins.github.io/
