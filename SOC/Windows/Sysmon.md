
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