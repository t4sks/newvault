[[#Vulnerability 1 - Brocken Object Levl Authorisation (BOLA)]]
[[#Vulnerability 2 - Brocken User Authentication (BUA)]]
[[#Vulnerability 3 - Excessive Data Exposure - чрезмерное раскрытие данных]]
[[#Vulnerability 4 - Lack of Resources & Rate Limiting - отсутствие ограничения по ресурсам]]
[[#Vulnerability 5 - Broken Function Level Authorization - когда сервер выдает пользователю доступ к функциям которые он не должен видеть или запускать, потому что проверка "может ли пользователь выполнить эту функцию" отсутствует или неверна]]

---
API(Application Programming Interface) - прослойка которая позволяет двум программным компонентам взаимодействовать друг с другом с помощью набора протоколов и определений

---
### Vulnerability 1 - Brocken Object Levl Authorisation (BOLA)
when server give the access to data by easy indentifire(exapmple: /user/1) anddont check can rhis account see another ID
Fix
Принцип - Авторизация ВСЕГДА должна быть на уровне объекта
1. Authorization  - проверка прав конкретного аутентифицированного аккаунта пользователя на конкретный объект или операцию
2. Нельзя полагаться на скрытые поля, предсказуемые ID, URL-переменные, клиентскую логику или отсутствие UI-элементов как защиту
Где реализовывать проверки: 
1.На сервере, в контроллере или в сервисном слое, ДО получения/возвращения данных
2.в REST/GraphQL - в обработчике запроса и/или в уровне доступа к данным(repository/ORM hooks)
Конкретная логика проверки
```http
// запрос: GET /users/{id}
user = authenticate_request(request)             // 401 если не аутентифицирован
target = db.find_user_by_id(request.params.id)   // 404 если нет объекта
if not is_authorized(user, 'read', target):
    return 403 Forbidden
return 200 OK with target.data
```
`is_authorized` - проверяет
- совпадение `user.id == target.owner_id`
- роль/политику 
- дополнительные условия - время доступа, связь между объектами

Model of authorization 
RBAC(Role-Based Access Control) - rights gives for roles, roles - accounts
ABAC(Attribute-Based Access Control) - права зависят от свойств субъекта, ресурса, среды
Policy-Based - centralized rules 

### Vulnerability 2 - Brocken User Authentication (BUA)
это когда система неправильно проверяет логин или пароль пользователя из за чего злоумышленник может войти как другой человек имея всего лишь часть данных
Исправление
Никогда не перекладывать на front разработчиков фильтрацию чувствительных данных
Переодически проверять ответы API чтобы убеидтся что они содержат только допустимые данные
Избегать использования общих методов `to_string()` и `to_json`
Тестировать API эндпоинты через разные сценарии и проверять не происходит ли утечка лишних данных

### Vulnerability 3 - Excessive Data Exposure - чрезмерное раскрытие данных
API дает сильно много информации,  вместо нужных полей может вернуть токен, или пароль

### Vulnerability 4 -  Lack of Resources & Rate Limiting - отсутствие ограничения по ресурсам
отсутствие ограничений и частоте запросов означает что API никак не контролирует сколько раз клиент может обращаться к серверу, и какого размера файлы он загружает. Это сильно нагружает сервер и может привести к отказу в обслуживании 

### Vulnerability 5 - Broken Function Level Authorization - когда сервер выдает пользователю доступ к функциям которые он не должен видеть или запускать, потому что проверка "может ли пользователь выполнить эту функцию" отсутствует или неверна 