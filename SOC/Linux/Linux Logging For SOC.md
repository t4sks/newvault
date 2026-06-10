# Working With Logs
Большинство логов событий записываются как текст. Но стандартные журналы Linux менее структурированы, поскольку в них отсутствуют коды событий и действуют строгие правила форматирования. 

Большинство Логов Linux хранятся в `/var/log`
Пример файла `/var/log/syslog` - файла системных событий

```shell
root@thm-vm:~$ cat /var/log/syslog | head 
[...] 
2025-08-13T13:57:49.388941+00:00 thm-vm systemd-timesyncd[268]: Initial clock synchronization to Wed 
2025-08-13 13:57:49.387835 UTC. 2025-08-13T13:59:39.970029+00:00 thm-vm systemd[888]: Starting dbus.socket - D-Bus User Message Bus Socket... 
2025-08-13T14:02:22.606216+00:00 thm-vm dbus-daemon[564]: [system] Successfully activated service 'org.freedesktop.timedate1' 
2025-08-13T14:05:01.999677+00:00 thm-vm CRON[1027]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1) [...]
```
## Фильтрация логов
При открытии файла вы увидите тысячи событий которые читает Syslog, но полезные для SOC только несколько типов. Именно поэтому вы должны фильтровать логи и сужать область поиска на столько на сколько это возможно. Например можно использовать `grep CRON` чтобы выделить все логи относящиеся к Cronjobs
```shell
# Or "grep -v CRON" to exclude "CRON" from results 
root@thm-vm:~$ cat /var/log/syslog | grep CRON 
2025-08-13T14:17:01.025846+00:00 thm-vm CRON[1042]: (root) CMD (cd / && run-parts --report /etc/cron.hourly) 
2025-08-13T14:25:01.043238+00:00 thm-vm CRON[1046]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1) 
2025-08-13T14:30:01.014532+00:00 thm-vm CRON[1048]: (root) CMD (date > mycrondebug.log)
```

## Исследование логов
Например если мы хотим найти сессии, логи но не знаем где их искать, но мы знаем что все логи хранятся в `/var/log` в тексте, мы можем просто использовать `grep` для поиска ключевых слов, например `login auth session`, во всех файлах
```shell
# List what's logged by your system (/var/log folder) 
root@thm-vm:~$ ls -l /var/log 
drwxr-xr-x 2 root root 4096 Aug 12 16:41 apt 
drwxr-x--- 2 root adm 4096 Aug 12 12:40 audit 
-rw-r----- 1 syslog adm 45399 Aug 13 15:05 auth.log 
-rw-r--r-- 1 root root 1361277 Aug 12 16:41 dpkg.log 
drwxr-sr-x+ 3 root systemd-journal 4096 Oct 22 2024 journal 
-rw-r----- 1 syslog adm 214772 Aug 13 13:57 kern.log 
-rw-r----- 1 syslog adm 315798 Aug 13 15:05 syslog [...]
# Search for potential logins across all logs (/var/log) 
root@thm-vm:~$ grep -R -E "auth|login|session" /var/log [...]
```

## Ограничения связанные с логами
В отличие от windows Linux позволяет легко изменять формат логов, уровень детализации и место их хранения в зависимости от дистрибутива
# Логи аутентификации
Первые самые часто используемые логи это логи аутентификации, они находятся в `/var/log/auth.log` (или в `/var/log/secure` в RHEL-based системах). Хотя его предполагает что он содержит события аутентификации, он также может хранить события управления пользователями, запущенные команды sudo и так далее.
Формат журнала представлен ниже:
![[Pasted image 20260610191441.png]]
## События входа и выхода из системы
Существует много способов аутентификации пользователей в Linux: локально, через ssh, используя sudo или su, или автоматически запуска задания крон. Каждый успешный вход и выход логируется и вы можете увидеть его отфильтровав события содержащие `session opened` и `session closed`
Локальные и удаленные входы:

```shell
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'session opened|session closed' 
# Local, on-keyboard login and logout of Bob (login:session) 
2025-08-02T16:04:43 thm-vm login[1138]: pam_unix(login:session): session opened for user bob(uid=1001) by bob(uid=0) 
2025-08-02T19:23:08 thm-vm login[1138]: pam_unix(login:session): session closed for user bob 
# Remote login examples of Alice (via SSH and then SMB) 
2025-08-04T09:09:06 thm-vm sshd[839]: pam_unix(sshd:session): session opened for user alice(uid=1002) by alice(uid=0) 
2025-08-04T12:46:13 thm-vm smbd[1795]: pam_unix(samba:session): session opened for user alice(uid=1002) by alice(uid=0)
```

Cron и Sudo входы

```shell
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'session opened|session closed' 
# Traces of some cron job launch running as root (cron:session) 
2025-08-06T19:35:01 thm-vm CRON[41925]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0) 
2025-08-06T19:35:01 thm-vm CRON[3108]: pam_unix(cron:session): session closed for user root 
# Carol running "sudo su" to access root (sudo:session) 
2025-08-07T09:12:32 thm-vm sudo: pam_unix(sudo:session): session opened for user root(uid=0) by carol(uid=1003)
```

В дополнение к логах входа SSH демон сохраняет логи о успешных и неуспешных входах. Эти логи лежат там же в `auth.log`, но имеют немного другой формат. 
Вот пример ssh логов 

```shell
root@thm-vm:~$ cat /var/log/auth.log | grep "sshd" | grep -E 'Accepted|Failed' 
# Common SSH log format: <is-successful> <auth-method> for <user> from <ip> 
2025-08-07T11:21:25 thm-vm sshd[3139]: Failed password for root from 222.124.17.227 port 50293 ssh2 
2025-08-07T14:17:40 thm-vm sshd[3139]: Failed password for admin from 138.204.127.54 port 52670 ssh2 
2025-08-09T20:30:51 thm-vm sshd[1690]: Accepted publickey for bob from 10.19.92.18 port 55050 ssh2: <key>
```
## Разные события 
Вы так же можете использовать тот же файл логов для того чтобы увидеть события связанные с управлением пользователями. Это легко если знать базовые команды управления пользователями. 

```shell
root@thm-vm:~$ cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\[' 
2023-02-01T11:09:55 thm-vm passwd[644]: password for 'ubuntu' changed by 'root' 
2025-08-07T22:11:11 thm-vm userdel[1887]: delete user 'oldbackdoor' 
2025-08-07T22:11:29 thm-vm useradd[1878]: new user: name=backdoor, UID=1002, GID=1002, shell=/bin/sh 
2025-08-07T22:11:54 thm-vm usermod[1906]: add 'backdoor' to group 'sudo' 
2025-08-07T22:11:54 thm-vm usermod[1906]: add 'backdoor' to shadow group 'sudo'
```

В зависимости от конфигурации системы и установленных пакетов мы можете столкнуться с интересными событиями. Например вы можете обнаружить команды запущенные с помощью sudo, что может помочь отследить вредоносные действия. В приведенном ниже примере пользователь "ubuntu" использовал sudo для остановки EDR чтения состояния брандмауэра и доступа к root 

```shell
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'COMMAND=' 
2025-08-07T11:21:49 thm-vm sudo: ubuntu : TTY=pts/0 ; [...] COMMAND=/usr/bin/systemctl stop edr 
2025-08-07T11:23:18 thm-vm sudo: ubuntu : TTY=pts/0 ; [...] COMMAND=/usr/bin/ufw status numbered 
2025-08-07T11:23:33 thm-vm sudo: ubuntu : TTY=pts/0 ; [...] COMMAND=/usr/bin/su
```

# Общие системные логи
Линукс отслеживает множество событий разбросанных по другим файлам в `/var/log`: kernel логи, network изменения, service или cron runs, package installation и многое другое. Их формат может отличаться от системы к системе, но самые популярные файлы: 

- `/var/log/kern.log` - логи ядра, сообщения и ошибки, полезные для большинства продвинутых исследований
- `/var/log/syslog (or /var/log/messages)` - сбор потока разных линукс событий
- `/var/log/dpkg.log (or /var/log/apt)` - менеджер пакетов в Debain
- `/var/log/dnf.log (or /var/log/yum.log)` менеджер пакетов в RHEL системах 

# Логи специальных приложений
Для эффективного выполнения работы SOC аналитика вы можете использовать определенные журналы, например Анализируйте журналы баз данных чтобы узнать какие запросы были выполнены, журналы электронной почты для расследования фишинга, журналы контейнеров для выявления аномалий и журналы веб сервера, чтобы узнать какие страницы были открыты, когда и кем. 
Ниже приведен пример журналов сервера nginx

```shell
root@thm-vm:~$ cat /var/log/nginx/access.log  
# Every log line corresponds to a web request to the web server 
10.0.1.12 - - [11/08/2025:14:32:10 +0000] "GET / HTTP/1.1" 200 3022 
10.0.1.12 - - [11/08/2025:14:32:14 +0000] "GET /login HTTP/1.1" 200 1056 
10.0.1.12 - - [11/08/2025:14:33:09 +0000] "POST /login HTTP/1.1" 302 112 
10.0.4.99 - - [11/08/2025:17:11:20 +0000] "GET /images/logo.png HTTP/1.1" 200 5432 10.0.5.21 - - [11/08/2025:17:56:23 +0000] "GET /admin HTTP/1.1" 403 104
```

# Bash History
Другой достаточно используемый ресурс это логи Bash, функция которая записывает каждую выполненную команду после нажатия на Enter. По умолчанию команды сначала хранятся в памяти на протяжении сессии и после записываются в папке юзера в `~/.bash_history` после выхода.
Пример содержимого файла: 

```shell
ubuntu@thm-vm:~$ cat /home/ubuntu/.bash_history 
echo "hello" > world.txt 
nano /etc/ssh/sshd_config 
sudo su 
ubuntu@thm-vm:~$ history 
1 echo "hello" > world.txt 
2 nano /etc/ssh/sshd_config 
3 sudo su 
4 ls -la /home/ubuntu 
5 cat /home/ubuntu/.bash_history 
6 history
```

# Системные вызовы
Всякий раз когда вам нужно открыть файл, создать процесс, получить доступ к камере или запросить любую другую службу ОС, вы совершаете определенный системный вызов. В Linux существует более 300 системных вызовов таких как `execve` для выполнения программы, ниже приведена блок схема как все это устроено: ![[Pasted image 20260610195709.png]]

По факту все современные EDR системы и инструменты логирования основаны на них, они отслеживают основные системные вызовы и записывают подробности в удобочитаемом формате. Поскольку у злоумышленников практически нет возможности обойти системные вызовы все что нам нужно сделать это выбрать системные вызовы которые хотите регистрировать и отслеживать
# Audit Daemon
Файл конфигурации правил для auditd лежит в `/etc/audit/rules.d/` который определяет какие системные вызовы отслеживать и какие фильтры применять
![[Pasted image 20260610200219.png]]
## Using Auditd
Вы можете отслеживать логи которые генерируются в реальном времени в `/var/log/audit/audit.log` но легче использовать `ausearch` команду, потому что она форматирует вывод для лучшей читаемости и поддерживает опции фильтра. Пример вывода с фильтром `proc_wget`

```shell
root@thm-vm:~$ ausearch -i -k proc_wget 
---- 
type=PROCTITLE msg=audit(08/12/25 12:48:19.093:2219) : proctitle=wget https://files.tryhackme.thm/report.zip 
type=CWD msg=audit(08/12/25 12:48:19.093:2219) : cwd=/root 
type=EXECVE msg=audit(08/12/25 12:48:19.093:2219) : argc=2 a0=wget a1=https://files.tryhackme.thm/report.zip 
type=SYSCALL msg=audit(08/12/25 12:48:19.093:2219) : arch=x86_64 syscall=execve [...] ppid=3752 pid=3888 auid=ubuntu uid=root tty=pts1 exe=/usr/bin/wget key=proc_wget
```

Приведенный выше терминал отображает лог одной команды "wget". Здесь auditd разделяет событие на четыре строки
PROCTITILE показывает командную строку процесса.
CWD показывает текущий рабочий каталог
а оставшиеся две строки показывают подробности системного вызова например: 

- `pid=3888, ppid=3752` PID процесса и PPID родительский идентификатор процесса
- `auid=ubuntu` пользователь. Аккаунт который был использован при входе, локально с клавиатуры или удаленно через ssh
- `uid=root` пользователь который запустил команду. Поле может быть разным с `auid` например если вы поменяли пользователя используя su или sudo 
- `tty=pts1` идентификатор сессии помогает различать события когда разные люди работают на одном Linux сервере
- `exe=/usr/bin/wget` абсолютный путь до исполняемого бинарника, часто используется для построения правил детектирования SOC
- `key=proc_wget` опциональный тэг спецификации для инженеров в auditd правилах который полезен для фильтрации событий

