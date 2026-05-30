---
tags: [soc, web, owasp, roadmap]
section: 02
---

# 02 - Web (OWASP)

[[00 - SOC Junior Roadmap (BiZone)]]

> **Лаба.** Подними **OWASP Juice Shop** (docker) и **DVWA** — намеренно уязвимые приложения. Атакуй их с Burp Suite (Community). Для систематической практики — **PortSwigger Web Security Academy** (бесплатно, лучшее в мире по вебу). У тебя база веба есть — здесь упор на атаки + как они видны в логах (детект-сторона для SOC).
>
> Друзья советуют OWASP 2017 (конкретнее по атакам). **Формат:** 📖 читать · 🛠 трогать · 💬 промпт мне.

---

## OWASP Top 10 — 2017 (приоритет)

- [ ] A1 Injection — SQLi, command injection, LDAP
  - [ ] SQLi виды: error/union/blind (boolean/time)/out-of-band
  - [ ] Детект: ошибки БД в логах, аномальные параметры, WAF
- [ ] A2 Broken Authentication — брутфорс, credential stuffing
- [ ] A3 Sensitive Data Exposure
- [ ] A4 XXE
- [ ] A5 Broken Access Control — IDOR, path traversal
- [ ] A6 Security Misconfiguration
- [ ] A7 XSS — reflected, stored, DOM
- [ ] A8 Insecure Deserialization
- [ ] A9 Known Vulnerabilities
- [ ] A10 Insufficient Logging & Monitoring (важно для SOC)

📖 **Читать:** Habr (OWASP) «OWASP Top 10 2017» https://habr.com/ru/companies/owasp/articles/342986/ ; «практический взгляд» https://habr.com/ru/companies/simplepay/articles/258499/ ; ctf.msk.ru с примерами https://ctf.msk.ru/p/owasp-top-10/

🛠 **Потрогать (Juice Shop / DVWA + Burp):**
```
1. docker run -p 3000:3000 bkimminich/juice-shop
2. Открой через Burp (proxy), перехвати запросы
3. SQLi: в поле логина введи ' OR 1=1-- — обойди аутентификацию
4. XSS: найди поле, вставь <script>alert(1)</script>
5. IDOR: поменяй id в URL/запросе, получи чужие данные
6. ДЕТЕКТ: посмотри логи приложения/прокси — как выглядит атака со стороны защитника
```
Задание: проэксплуатируй SQLi и XSS в Juice Shop. Потом найди эти запросы в логах и опиши, по каким признакам SOC их детектит.

💬 **Промпт мне:** «Веду расследование: в access.log вижу запросы с ' OR 1=1 и UNION SELECT. Объясни, что за атака, на каком этапе (успех/попытка), и какие ещё признаки SQLi искать в логах».

---

## OWASP Top 10 — 2021 (знать обе)

- [ ] A01 Broken Access Control, A02 Cryptographic Failures, A03 Injection, A04 Insecure Design, A05 Security Misconfiguration, A06 Vulnerable Components, A07 Identification/Auth Failures, A08 Software/Data Integrity, A09 Logging/Monitoring Failures, A10 SSRF

📖 **Читать:** Habr «как менялись риски 2017 vs 2021» https://habr.com/ru/companies/webmonitorx/articles/974954/ ; OWASP офиц. https://owasp.org/www-project-top-ten/

💬 **Промпт мне:** «Сравни OWASP 2017 и 2021: что объединили, что добавили (Insecure Design, SSRF), почему Broken Access Control стал A01. Таблицей соответствия».

---

## Дополнительные веб-атаки

- [ ] CSRF, SSRF (cloud metadata), file upload → web shell, LFI/RFI, path traversal, HTTP smuggling

📖 **Читать:** PortSwigger Academy (по каждой теме отдельная лаба) https://portswigger.net/web-security

🛠 **Потрогать:** Пройди на PortSwigger по 2-3 лабы на SQLi, XSS, SSRF, Access Control. Они бесплатные и с проверкой.

💬 **Промпт мне:** «Объясни SSRF и почему он критичен в облаке (169.254.169.254, кража токенов). Дай пример атаки и признаки в логах сервера».

---

## Детект веб-атак (SOC-фокус)

- [ ] Анализ access.log (Apache/Nginx): аномальные URL, User-Agent, частота
- [ ] Признаки сканирования (dirbuster, nikto, sqlmap)
- [ ] Признаки web shell (новые .php/.jsp/.aspx, обращения к ним)
- [ ] WAF-логи, корреляция

🛠 **Потрогать:**
```bash
# натрави сканер на Juice Shop и посмотри логи:
nikto -h http://localhost:3000
# в логах увидишь шквал запросов с User-Agent сканера — это сигнатура
```

💬 **Промпт мне:** «Дай мне признаки в access.log для: сканирования (nikto/dirb), SQLi-попыток, заливки web shell, брутфорса. По каждому — пример строки лога и grep для поиска».

---

## Progress

- [ ] Раздел Web пройден полностью
- [ ] Проэксплуатированы SQLi/XSS/IDOR в Juice Shop
- [ ] Пройдено 5+ лаб PortSwigger
- [ ] Разобран детект атак в логах

---

## 🔭 Связь со следующими шагами

- **→ AppSec (твоя цель!):** этот раздел — прямой фундамент. Дальше: вся PortSwigger Academy, bug bounty (HackerOne/Bugcrowt), книга «Web Application Hacker's Handbook», secure code review. Уже сейчас копай глубже остальных.
- **→ Blueteam:** детект веб-атак в логах, WAF-тюнинг — веб-часть SOC.
- **→ Инфрапентест:** web shell и RCE — точка входа во внутреннюю сеть (связь с [[01 - Linux]] и pivoting).

---

## 🧪 Тест для повторения

> [!question]- 1. Виды SQL-инъекций и детект в логах.
> Error-based, union-based, blind (boolean/time), out-of-band. Детект: ошибки СУБД в логах, аномальные символы (' UNION SLEEP --), всплеск 500-х, срабатывания WAF, User-Agent sqlmap.

> [!question]- 2. Три типа XSS и различие.
> Reflected — payload в запросе, отражается сразу. Stored — сохраняется на сервере, срабатывает у других. DOM-based — в браузере через небезопасный JS, сервер не участвует.

> [!question]- 3. Что такое IDOR, к какой категории OWASP?
> Insecure Direct Object Reference — доступ к чужому объекту подменой ID (?id=123→124). Broken Access Control (A5 в 2017 / A01 в 2021).

> [!question]- 4. SSRF и почему критичен в облаке?
> Server-Side Request Forgery — заставить сервер сделать запрос к внутреннему ресурсу. В облаке — доступ к metadata (169.254.169.254) → кража токенов инстанса. A10 в OWASP 2021.

> [!question]- 5. Признаки web shell в access.log.
> Новые исполняемые файлы (.php/.jsp/.aspx) в загрузочных каталогах, POST/GET к ним с необычными параметрами (cmd=), доступ с одного IP, нетипичные исходящие от веб-сервера.

> [!question]- 6. Отличие OWASP 2021 от 2017 структурно.
> 2021 укрупнил категории, добавил Insecure Design и Software/Data Integrity Failures, вынес SSRF (A10), поднял Broken Access Control на A01. 2017 — конкретнее по классам атак.

> [!question]- 7. Что такое CSRF и чем отличается от XSS?
> CSRF — принуждение браузера жертвы выполнить действие на сайте, где она авторизована (через её куки). XSS — выполнение чужого JS в браузере жертвы. CSRF использует доверие сайта к пользователю, XSS — доверие пользователя к сайту.

> [!question]- 8. Что такое path traversal / LFI?
> Path traversal — выход за пределы каталога через ../ для чтения произвольных файлов (../../etc/passwd). LFI (Local File Inclusion) — включение/исполнение локального файла приложением. RFI — то же с удалённым файлом.

> [!question]- 9. Что такое XXE?
> XML External Entity — атака на XML-парсер: через внешние сущности можно читать файлы сервера, делать SSRF, иногда RCE. Возникает при небезопасной обработке XML с включёнными external entities.

> [!question]- 10. Почему A10 2017 (Insufficient Logging) важен именно для SOC?
> Без достаточного логирования и мониторинга атаки не детектируются — SOC «слепнет». Это единственный пункт OWASP про защиту, а не уязвимость кода; прямо про работу аналитика.

> [!question]- 11. Что такое insecure deserialization?
> Небезопасная десериализация недоверенных данных: подсунув вредоносный сериализованный объект, атакующий может добиться RCE, privesc, инъекций. A8 в 2017.

> [!question]- 12. Как выглядит брутфорс/credential stuffing в логах?
> Множество POST на endpoint логина с разными паролями (брутфорс) или разными парами логин:пароль из утечек (stuffing), часто с одного IP или ботнета, высокая частота, всплеск 401/403.

---

## 📚 Все источники раздела (сводно)

- Habr (OWASP) — Top 10 2017: https://habr.com/ru/companies/owasp/articles/342986/
- Habr — практический взгляд: https://habr.com/ru/companies/simplepay/articles/258499/
- Habr — 2017 vs 2021: https://habr.com/ru/companies/webmonitorx/articles/974954/
- ctf.msk.ru — OWASP с примерами: https://ctf.msk.ru/p/owasp-top-10/
- OWASP офиц.: https://owasp.org/www-project-top-ten/
- PortSwigger Web Security Academy (практика!): https://portswigger.net/web-security
- OWASP Juice Shop: https://github.com/juice-shop/juice-shop
- DVWA: https://github.com/digininja/DVWA
