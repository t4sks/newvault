
> [!NOTE] Sysmon
> Это системная служба windows которая остается активной после перезагрузок, отслеживает системную активность в журнал событий Windows. Он предоставляет детальную информацию о создании процессов, сетевых подключений и изменениях времени создания файлов. Собирая и генерируя события через windows event collection или агенты SIEM и анализируя их, можно выявлять вредоносную активность и понимать как злоумышленники действуют в сети

Sysmon собирает детальные логи а так же трассировку событий, что помогает выявлять аномалии в среде. Sysmon чаще всего используется совместно с SIEM или другими решениями для парсинга логов.

События Sysmon хранятся по пути: `Applications and Services Logs/Microsoft/Windows/Sysmon/Operational`
# EventID 1 создание процесса
Это событие отслеживает все созданные процессы. Используется для поиска известных подозрительных процессов или процессов с типами которые которые являются аномалией. 
Событие задействует XML теги `CommandLine` и `Image`

```
<RuleGroup name="" groupRelation="or">  
<ProcessCreate onmatch="exclude">  
  <CommandLine condition="is">C:\Windows\system32\svchost.exe -k appmodel -p -s camsvc</CommandLine>  
</ProcessCreate>  
</RuleGroup>
```

Код вышек указывает EventID для захвата и условие. В данном случае процесс `svchost.exe`
# EventID 3 сетевое подключение
Событие сетевого подключение отслеживает события происходящие удаленно. Включает файла и источники подозрительных бинарников, а так же открытые порты 
Задействует теги `Image` и `DestinationPort`

```
<RuleGroup name="" groupRelation="or">  
<NetworkConnect onmatch="include">  
  <Image condition="image">nmap.exe</Image>  
  <DestinationPort name="Alert,Metasploit" condition="is">4444</DestinationPort>  
</NetworkConnect>  
</RuleGroup>
```

Фрагмент выше задействует два способа выявления подозрительной активности. Первый идентифицирует файлы передаваемые через открытые порты, в данном случае конкретные это NMAP. Второй выявляет открытые порты в частности 4444 который часто используется Metasploit. При выполнении условия создается событие которое в идеале триггерит алерт для расследования SOC

# EventID 7 Загрузка образа
Это событие отслеживает DLL, загружаемые процессами, полезно при охоте на DLL Injection и DLL Hijacking. Рекомендуется соблюдать осторожность при использовании этого EventID, так как он дает самую высокую нагрузку на систему. 
Задействует XML теги `Image` `Signed` `ImageLoaded` `Signature`

```
<RuleGroup name="" groupRelation="or"> 
	<ImageLoad onmatch="include"> 
		<ImageLoaded condition="contains">\Temp\</ImageLoaded> 
	</ImageLoad> 
</RuleGroup>
```

Фрагмент выше отслеживает любые DLL загруженные из директории `\Temp\`. Загрузка DLL из этой директории считается аномалией и подлежит расследованию

# EventID 8 CreateRemoteThread
EventID CreateRemoteThread это событие которое отслеживает процессы внедряющие код в другие процессы. Функция `CreateRemoteThread` используется в легитимных задачах и приложениях, однако может применяться вредоносным ПО для сокрытия активности. Задействует XML теги `SourceImage`, `TargetImage`, `StartAddress` и `StartFunction`

```
<RuleGroup name="" groupRelation="or">
    <CreateRemoteThread onmatch="include">
        <StartAddress name="Alert,Cobalt Strike" condition="end with">0B80</StartAddress>
        <SourceImage condition="contains">\</SourceImage>
    </CreateRemoteThread>
</RuleGroup>
```

Фрагмент выше показывает два способа мониторинга CreateRemoteThread. Первый анализирует адрес памяти на конкретное конечное условие - индикатор маяка CobaltStrike. Второй ищет внедренные процессы без родительского процесса, что является аномалией и требует расследования
# EventID 11 создание файла 
Это событие отслеживает создание или перезапись файла на конечной точке. Используется для идентификации имен и сигнатур файлов, записанных на диск. 
Задействует XML теги `TargetFilename`

```
<RuleGroup name="" groupRelation="or">
    <FileCreate onmatch="include">
        <TargetFilename name="Alert,Ransomware" condition="contains">HELP_TO_SAVE_FILES</TargetFilename>
    </FileCreate>
</RuleGroup>
```

Фрагмент кода это пример для мониторинга программ вымогателей (ransomware). Это лишь один из множества вариантов применения EventID 11
# EventID 12/13/14 События реестра 
Эти события отслеживают изменения и модификации реестра. Вредоносная активность через реестр может включать закрепление в системе и злоупотребление учетными записями. 
Задействует `TargetObject` XML тег

```
<RuleGroup name="" groupRelation="or">
    <RegistryEvent onmatch="include">
        <TargetObject name="T1484" condition="contains">Windows\System\Scripts</TargetObject>
    </RegistryEvent>
</RuleGroup>
```

Фрагмент выше отслеживает объекты реестра в директории `Windows\System\Scripts` - распространенное место размещения скриптов злоумышленниками для закрепления
# EventID 15 FileCreateStreamHash
Это событие отслеживает файлы созданные в альтернативном потоке данных (Alternative Data Stream). Распространенная техника сокрытия вредоносного ПО. 
Задействует XML тег `TargetFilename`

```
<RuleGroup name="" groupRelation="or">
    <FileCreateStreamHash onmatch="include">
        <TargetFilename condition="end with">.hta</TargetFilename>
    </FileCreateStreamHash>
</RuleGroup>
```

Фрагмент выше отслеживает файлы с расширением `.hta`, помешенные в альтернативный поток данных 
# EventID 22 DNS-событие
Это событие логирует все DNS запрос и события для анализа. Стандартный подход это исключить все недоверенные домены, генерирующие значительный шум в среде. После устранения шума можно искать DNS аномалии. 
Задействует XML тег `QueryName`

```
<RuleGroup name="" groupRelation="or">
    <DnsQuery onmatch="exclude">
        <QueryName condition="end with">.microsoft.com</QueryName>
    </DnsQuery>
</RuleGroup>
```

Фрагмент выше исключает все события с запросом `.microsoft.com` устраняя шум в среде.

# Hunting Metasploit
[Malware Common Ports Spreadsheet](https://docs.google.com/spreadsheets/d/17pSTDNpa0sf6pHeRhusvWG6rThciE8CsXTSlDUAZDy) 
Ищем подозрительные образы, порты которые обычно использует Mimikatz то есть 4444 и 5555, так же смотрим на местоположение IP
Пример для поиска портов
```
Get-WinEvent -Path <Path to Log> -FilterXPath '*/System/EventID=3 and */EventData/Data[@Name="DestinationPort"] and */EventData/Data=4444'
```
# Detecting Mimikatz
Mimiratz широко известный и часто используемый инструмент для извлечения учетных данных из оперативной памяти, а так же выполнения других действий после компрометации. Чаще всего Mimikatz используют для получения содержимого процесса LSASS

Для обнаружения Mimikatz можно искать 

- Создание файла mimikatz
- Запуск этого файла из процесса с повышенными привилегиями
- Создание удаленного потока 
- Процессы которые создает Mimikatz

Антивирусы обычно знают все сигнатуры mimikatz. Однако злоумышленники могут обфусцировать программу или использовать дропперы, чтобы незаметно доставить программу на устройство. 
## Обнаружение создания файла mimikatz
Первый метод заключается в простом поиске файлов с названием mimikatz. Простая техника которая поможет найти то что пропустил AV. При расследовании атак высокого уровня обычно применяются более сложные методы, например анализ поведения LSASS, однако поиск по имени файла может оказаться полезным 
## Поиск аномального поведения LSASS
Мы можем использовать `EventID 10` для поиска подозрительного поведения LSASS

Если какое либо приложение получает доступ к LSASS это событие будет зафиксировано. Подобная активность может свидетельствовать о попытке извлечения учетных данных из памяти, что часто связано с использованием Mimikatz или других инструментов для кражи учетных данных.

Если к процессу LSASS обращается процесс отличный от `svchost.exe` такое поведение следует считать подозрительным и следует провести дополнительное расследование

Чтобы упростить поиск подобных событий можно настроить фильтр отображающий только процессы отличные от svchost.exe

Так же Sysmon предоставляет дополнительную информацию, которая помогает расследованию, например полный путь к исполняемому файлу процесса получившего доступ к LSASS
# Hunting Malware Overview
Здесь рассмотрим два типа RATs и backdors. 

RATs или Remote Access Trojans используют множество похожих пейлоадов для получения удаленного доступа к машине. RATs обычно так же использует  другие методы обхода антивирусной защиты и обнаружения, что отличает их от других полезных нагрузок, таких как MSFVenom. RAT программы обычно используют клиент-серверную модель и имеют интерфейс для простого администрирования пользователем. 
Примером RAT-программы являются Xeexe и Quasar. Для обнаружения нам необходимо определить, какое именно вредоносное ПО нам сначала необходимо определить какое именно вредоносное ПО мы хотим обнаружить и выявить способы изменения конфигурационных файлов. 
Это называется поиском на основе гипотез. 
## Hunting Rats и C2 серверов 
первый метод поиска вредоносного ПО очень похож на поиск активности Metasploit 

Можно создать или изменить конфигурационный файл Sysmon, который будет отслеживать подозрительные сетевые соединения, например подключения к известным вредоносным портам

Используя список известных подозрительных портов и добавляя их к правила мониторинга, можно построить методику поиска, которая позволяет:

- Обнаружить злоумышленников по журналам событий
- Затем использовать сетевые дампы (packet captures) или другие методы анализа для продолжения расследования
# Hunting Persistence
Закрепления используется для сохранения доступа к машине после ее компрометации. Существует множество способов закрепиться в системе.
Ниже рассмотрим модификации реестра и скриптах автозапуска. 
Искать можно с помощью событий создания файлов EventID 11 и модификации реестра EventID 12/13/14
## Hunting Startup Persistence 
Сначала рассмотрим детекты когда файл помещается в `\Startup\` или `\Start Menu`. Ниже фрагмент конфигурации обеспечивающий трассировку событий для этой техники

```
<RuleGroup name="" groupRelation="or"> <FileCreate onmatch="include"> <TargetFilename name="T1023" condition="contains">\Start Menu</TargetFilename> <TargetFilename name="T1165" condition="contains">\Startup\</TargetFilename> </FileCreate> </RuleGroup>
```

Любые изменения связанные с автозагрузкой подлежат расследованию
## Hunting Registry Key Persistence
Рассмотрим вариант модификации реестра, при котором скрипт помещается в `CurrentVersion\Windows\Run`
# Evasion Techniques Overview
Авторы вредоносного ПО используют различные техники для обхода антивирусной защиты и систем обнаружения. 
Примеры: Alternate Data Streams (ADS), инъекции, маскировка, упаковка/сжатие, перекомпиляция, обфускация, техники анти-реверсинга. 

Alternate Data Stream - malware скрывает файлы от стандартной инспекции, сохраняя их в потоке, отличном от `$DATA`. Sysmon фиксирует создание и обращение к потокам через соответствующий EventID, что позволяет охотиться за malware использующей ADS

Техники инъекций бывают нескольких видов: 

- Перехват потока (Thread Hijacking)
- PE-инъекция
- DLL инъекции
- и другие методы

DLL инъекция - это перезапись легитимной DLL, используемой приложением, путем внедрения в нее вредоносного кода.
## Hunting Alternate Data Streams
используется EventID 15 - Sysmon хэшируется и логирует любые NTFS потоки описанные в конфигурационном файле. 
Пример файла который отслеживает `.hta` и `.bat` файлы

```
<RuleGroup name="" groupRelation="or">
  <FileCreateStreamHash onmatch="include">
    <TargetFilename condition="contains">Downloads</TargetFilename>
    <TargetFilename condition="contains">Temp\7z</TargetFilename>
    <TargetFilename condition="ends with">.hta</TargetFilename>
    <TargetFilename condition="ends with">.bat</TargetFilename>
  </FileCreateStreamHash>
</RuleGroup>
```

![[Pasted image 20260621182039.png]]

Видно что `not_malcious.exe` содержит скрытый поток `:malware:$DATA` - именно так malware прячет данные внутри легитимного файла
## Обнаружение удаленных потоков (Remote Threads)
Злоумышленники часто создают удаленные потоки в сочетании с другими техниками. Удаленные потоки создаются через Windows API `CreateRemoteThread` и могут быть доступны через `OpenThread` и `ResumeThread`. Это используется в DLL инъекции, Thread Hijacking и Process Hollowing

Используется Sysmon EventID 8.

![[Pasted image 20260621182638.png]]

Тут видно что `powershell.exe` создает удаленный поток и получает доступ к `notepad.exe`