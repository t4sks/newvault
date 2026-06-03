# Nmap сканирование
Nmap умеет сканировать 
- TCP connect скан
- SYN скан
- UDP скан

Для выявления активности очень важно понимать как работает Nmap, однако без использования фильтров невозможно понять детали сканирования
## TCP флаги для поиска в nmap

| **Notes**                                                   | **Wireshark Filters**                             |
| :---------------------------------------------------------- | :------------------------------------------------ |
| Global search                                               | `tcp`<br>`udp`                                    |
| Only SYN flag                                               | `tcp.flags == 2`                                  |
| SYN flag is set. The rest of the bits are not important     | `tcp.flags.syn == 1`                              |
| Only ACK flag                                               | `tcp.flags == 16`                                 |
| ACK flag is set. The rest of the bits are not important     | `tcp.flags.ack == 1`                              |
| Only SYN, ACK flags                                         | `tcp.flags == 18`                                 |
| SYN and ACK are set. The rest of the bits are not important | `(tcp.flags.syn == 1) and (tcp.flags.ack == 1)`   |
| Only RST flag                                               | `tcp.flags == 4`                                  |
| RST flag is set. The rest of the bits are not important     | `tcp.flags.reset == 1`                            |
| Only RST, ACK flags                                         | `tcp.flags == 20`                                 |
| RST and ACK are set. The rest of the bits are not important | `(tcp.flags.reset == 1) and (tcp.flags.ack == 1)` |
| Only FIN flag                                               | `tcp.flags == 1`                                  |
| FIN flag is set. The rest of the bits are not important     | `tcp.flags.fin == 1`                              |
## TCP Connect Scans
Краткое описание сканирования:
- Основывается на трехстороннем рукопожатии(необходимо завершить процесс рукопожатия)
- Обычно выполняется с помощью команды nmap -sT
- Используется непривилегированными пользователями
- Обычно размер окна превышает 1024 байта, поскольку запрос ожидает некоторые данные из за особенностей протокола

| **Open TCP Port**                        | **Open TCP Port**                                          | **Closed TCP Port**         |
| ---------------------------------------- | ---------------------------------------------------------- | --------------------------- |
| - SYN --><br>- <-- SYN, ACK<br>- ACK --> | - SYN --><br>- <-- SYN, ACK<br>- ACK --><br>- RST, ACK --> | - SYN --><br>- <-- RST, ACK |
Ниже приведены скриншоты с примерами трехстороннего рукопожатия при открытом и закрытом TCP портах
Open TCP Port (connect)
![[Pasted image 20260603201720.png]]

Closed TCP Port
![[Pasted image 20260603201801.png]]
Приведенные выше изображения иллюстрируют закономерности в изолированном трафике, однако, обнаружить эти закономерности в больших файлах захвата не всегда легко, поэтому аналитикам необходимо использовать универсальный фильтр для просмотра исходных аномальных закономерностей, после чего будет проще сосредоточиться на конкретной точке трафика. Приведенный фильтр показывает закономерности сканирования TCP connect в файле захвата
#### Фильтр для обнаружения такого скана
```
tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024
```

![[Pasted image 20260603202200.png]]

## SYN Scans
TCP SYN Scan кратко:

- Не требует трехстороннего рукопожатия
- Обычно выполняется с помощью nmap -sS
- Используется привилегированными пользователями
- Обычно имеет размер меньший или равный 1024 байтам поскольку запрос не завершен и не ожидает получения данных

| **Open TCP Port**                      | **Close TCP Port**         |
| -------------------------------------- | -------------------------- |
| - SYN --><br>- <-- SYN,ACK<br>- RST--> | - SYN --><br>- <-- RST,ACK |
Open TCP Port (SYN)
![[Pasted image 20260603202712.png]]

Closed TCP Port (SYN)
![[Pasted image 20260603202734.png]]

#### Фильтр который показывает такой скан
```
tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024
```
![[Pasted image 20260603202902.png]]
## UDP Scan
Краткое описание UDP сканирования

- Не требует процесса подтверждения соединения
- Нет запроса на открытие портов
- Сообщение об ошибке ICMP для закрытых портов
- Обычно исполняется с помощью команды nmap -sU

| **Open UDP Port** | **Closed UDP Port**                                                                            |
| ----------------- | ---------------------------------------------------------------------------------------------- |
| - UDP packet -->  | - UDP packet --><br>- ICMP Type 3, Code 3 message. (Destination unreachable, port unreachable) |
закрытый порт 69 и открытый порт 68
![[Pasted image 20260603203249.png]]
На изображении показано что закрытый порт возвращает пакет с ошибкой ICMP, Сообщение об ошибке ICMP использует исходный запрос в качестве инкапсулированных данных, чтобы показать источник/причину пакета. После разворачивания раздела ICMP в панели сведений о пакете вы увидите инкапсулированные данные и исходный запрос, как показано на изображении ниже![[Pasted image 20260603203535.png]]
#### Фильтр для нахождения UDP сканирования
```
icmp.type==3 and icmp.code==3
```
![[Pasted image 20260603203633.png]]

# ARP Poisoning/Spoofing (A.K.A. Man In The Middle Attack)
ARP Poisoning(также известная как подмена ARP пактов или атака "Человек по середине") - это тип атаки, который включает в себя подавление/манипулирование(jamming/manipulating) сетью путем отправки вредоносных ARP пакетов а шлюз по умолчанию. Конечная цель это манипулировать таблицей IP адресов и MAC адресов и перехватывать трафик целевого хоста

Существует множество инструментов для проведения ARP атак. Однако суть атаки статична, поэтому обнаружить такую атаку легко зная алгоритм работы ARP и используя навыки работы с WireShark

Анализ ARP вкратце:

- Работает в локальной сети
- Обеспечивает связь между MAC адресами не является безопасным протоколом
- Не является маршрутизируемым протоколом
- Не имеет функций аутентификации
- Распространенные шаблоны: запрос и ответ, объявление и пакеты с широковещательной рассылкой

| **Notes**                                              | **Wireshark filter**                                                     |
| ------------------------------------------------------ | ------------------------------------------------------------------------ |
| Global search                                          | `arp`                                                                    |
| **"ARP" options for grabbing the low-hanging fruits:** |                                                                          |
| Opcode 1: ARP requests.                                | `arp.opcode == 1`                                                        |
| Opcode 2: ARP responses.                               | `arp.opcode == 2`                                                        |
| **Hunt:**Arp scanning                                  | `arp.dst.hw_mac==00:00:00:00:00:00`                                      |
| **Hunt:**Possible ARP poisoning detection              | `arp.duplicate-address-detected or arp.duplicate-address-frame`          |
| **Hunt:**Possible ARP flooding from detection:         | `((arp) && (arp.opcode == 1)) && (arp.src.hw_mac == target-mac-address)` |
Далее рассмотрим легитимный трафик ARP 
Легитимный ARP request 
![[Pasted image 20260603210310.png]]
легитимный ARP reply
![[Pasted image 20260603210338.png]]

Подозрительная означает наличие двух разных ARP ответов для определенного IP адреса. В этом случае вкладка экспертная информация в wireshark предупреждает аналитика, однако она показывает только второе появление дублирующегося значения, чтобы подчеркнуть конфликт. Поэтому задача аналитика состоит в том, чтобы отличить вредоносный пакет от легитимного. Возможный случай ARP spoofinga показан на рисунке ниже:
![[Pasted image 20260603210742.png]]


| **Notes**                      | **Detection Notes**                                                                                                          | **Findings**                                            |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| Possible IP address match.     | 1 IP address announced from a MAC address.                                                                                   | - MAC: 00:0c:29:e2:18:b4<br>- IP: 192.168.1.25          |
| Possible ARP spoofing attempt. | 2 MAC addresses claimed the same IP address (192.168.1.1).  <br>The " 192.168.1.1" IP address is a possible gateway address. | - MAC1: 50:78:b3:f3:cd:f4<br>- MAC 2: 00:0c:29:e2:18:b4 |
| Possible ARP flooding attempt. | The MAC address that ends with "b4" claims to have a different/new IP address.                                               | - MAC: 00:0c:29:e2:18:b4<br>- IP: 192.168.1.1           |
| Possible ARP flooding attempt. | The MAC address that ends with "b4" crafted multiple ARP requests against a range of IP addresses.                           | - MAC: 00:0c:29:e2:18:b4<br>- IP: 192.168.1.xxx         |
![[Pasted image 20260603211722.png]]

На данном примере видно наличие аномалии. Происходит поток ARP запросов, от MAC адреса который оканчивается на `b4` сформировал множество ARP запросов с IP адресом 192.168.1.25