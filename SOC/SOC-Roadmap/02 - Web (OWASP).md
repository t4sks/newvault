---
tags: [soc, web, owasp, roadmap]
section: 02
---

# 02 - Web (OWASP)

[[00 - SOC Junior Roadmap (BiZone)]]

> Друзья советуют смотреть OWASP Top 10 **2017** (на ревизию старше 2021), там конкретнее по веб-атакам.

## OWASP Top 10 — 2017 (приоритет)

- [ ] A1 Injection — SQLi (классика), command injection, LDAP injection
  - [ ] SQLi виды: error-based, union-based, blind (boolean/time), out-of-band
  - [ ] Детект: аномальные параметры, ошибки БД в логах, WAF-срабатывания
- [ ] A2 Broken Authentication — брутфорс, credential stuffing, слабые сессии
- [ ] A3 Sensitive Data Exposure
- [ ] A4 XML External Entities (XXE)
- [ ] A5 Broken Access Control — IDOR, path traversal, privilege escalation
- [ ] A6 Security Misconfiguration
- [ ] A7 Cross-Site Scripting (XSS) — reflected, stored, DOM-based
- [ ] A8 Insecure Deserialization
- [ ] A9 Using Components with Known Vulnerabilities
- [ ] A10 Insufficient Logging & Monitoring (важно для SOC)

## OWASP Top 10 — 2021 (для сравнения, знать обе)

- [ ] A01 Broken Access Control
- [ ] A02 Cryptographic Failures
- [ ] A03 Injection
- [ ] A04 Insecure Design
- [ ] A05 Security Misconfiguration
- [ ] A06 Vulnerable and Outdated Components
- [ ] A07 Identification and Authentication Failures
- [ ] A08 Software and Data Integrity Failures
- [ ] A09 Security Logging and Monitoring Failures
- [ ] A10 Server-Side Request Forgery (SSRF)

## Дополнительные веб-атаки

- [ ] CSRF
- [ ] SSRF (и связь с pivoting/cloud metadata)
- [ ] File upload bypass → web shell
- [ ] LFI / RFI (Local/Remote File Inclusion)
- [ ] Path traversal `../../etc/passwd`
- [ ] HTTP request smuggling

## Детект веб-атак (SOC-фокус)

- [ ] Анализ access.log (Apache/Nginx): аномальные URL, User-Agent, частота
- [ ] Признаки сканирования (dirbuster, nikto, sqlmap в логах)
- [ ] Признаки web shell (новые .php/.jsp/.aspx, обращения к ним)
- [ ] WAF-логи, корреляция

## Practice

- [ ] PortSwigger Web Security Academy (бесплатно, лучшая практика)
- [ ] DVWA / Juice Shop локально

## Progress

- [ ] Раздел Web пройден полностью

---

## 🧪 Тест для повторения

> [!question]- 1. Назови виды SQL-инъекций и как детектить в логах.
> Error-based, union-based, blind (boolean / time-based), out-of-band. Детект: ошибки СУБД в логах приложения, аномальные символы в параметрах (`'`, `UNION`, `SLEEP`, `--`), всплеск 500-х, срабатывания WAF, характерные User-Agent (sqlmap).

> [!question]- 2. Три типа XSS и их различие.
> Reflected — payload в запросе, отражается в ответе сразу. Stored — payload сохраняется на сервере и срабатывает у других пользователей. DOM-based — выполнение в браузере через небезопасную работу JS с DOM, сервер не участвует.

> [!question]- 3. Что такое IDOR, к какой категории OWASP относится?
> Insecure Direct Object Reference — доступ к чужому объекту подменой идентификатора (напр. `?id=123` → `id=124`). Относится к Broken Access Control (A5 в 2017 / A01 в 2021).

> [!question]- 4. Что такое SSRF и почему критичен в облаке?
> Server-Side Request Forgery — заставить сервер сделать запрос к внутреннему ресурсу. В облаке — доступ к metadata-сервису (169.254.169.254) → кража токенов/кредов инстанса. A10 в OWASP 2021.

> [!question]- 5. Признаки web shell в access.log.
> Появление новых исполняемых файлов (.php/.jsp/.aspx) в загрузочных каталогах, последующие POST/GET к ним с необычными параметрами (cmd=, нестандартный User-Agent), доступ к файлу с одного IP, нетипичные исходящие соединения от веб-сервера.

> [!question]- 6. Главное отличие OWASP Top 10 2021 от 2017 (структурно).
> 2021 укрупнил категории (объединил, добавил «Insecure Design», «Software/Data Integrity Failures», вынес SSRF отдельно), Broken Access Control поднялся на A01. 2017 даёт более конкретные классы атак — поэтому друзья советуют его для подготовки.

---

## 📚 Источники для подготовки

**OWASP Top 10 — разбор на русском**
- Habr (OWASP) — OWASP Top 10 2017 (официальный блог, версия от друзей): https://habr.com/ru/companies/owasp/articles/342986/
- Habr — OWASP TOP-10: практический взгляд на безопасность веб-приложений: https://habr.com/ru/companies/simplepay/articles/258499/
- Habr — OWASP TOP 10 Project: введение (краткий разбор всех пунктов): https://habr.com/ru/company/gaz-is/blog/415283/
- Habr — OWASP Top Ten: как менялись риски (сравнение 2017 vs 2021): https://habr.com/ru/companies/webmonitorx/articles/974954/
- ctf.msk.ru — OWASP Top 10 с примерами атак и защитой: https://ctf.msk.ru/p/owasp-top-10/

**Официальные источники**
- OWASP Top 10 (актуальная редакция): https://owasp.org/www-project-top-ten/

**Практика (must-do)**
- PortSwigger Web Security Academy (бесплатные лабы по каждой уязвимости): https://portswigger.net/web-security
