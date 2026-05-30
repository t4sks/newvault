---
tags: [soc, activedirectory, kerberos, ntlm, roadmap]
section: 09
---

# 09 - Active Directory и атаки

[[00 - SOC Junior Roadmap (BiZone)]]

> **Лаба.** Подними **Windows Server 2022** (eval 180 дней), сделай контроллер домена (роль AD DS, `dcpromo`/Server Manager), создай домен типа `corp.local`. Добавь 1-2 клиентских Windows 10 в домен. Это твоя мини-AD. Для атак — Kali с **impacket** + **BloodHound**. Snapshot после настройки домена.
>
> Это самый частый вектор в корпоративе и приоритет №4 в твоём плане. Учим связкой **атака → детект**.
>
> **Формат:** 📖 читать · 🛠 трогать · 💬 промпт мне.

---

## Основы AD

- [ ] Domain, Forest, Tree, OU, Domain Controller
- [ ] Объекты: User, Computer, Group, GPO
- [ ] LDAP — протокол доступа к каталогу
- [ ] SYSVOL, NTDS.dit (`C:\Windows\NTDS\NTDS.dit` — база AD со всеми хешами)
- [ ] krbtgt — ключ ко всему домену
- [ ] SPN (Service Principal Name)

📖 **Читать:** codeby «Пентест AD: разведка для новичков» https://codeby.net/threads/active-directory-ot-passivnoi-razvedki-do-pervogo-shella-posobiye-dlya-nachinayushchikh.85943/ ; merionet «Что такое AD» https://wiki.merionet.ru/articles/cto-takoe-kerberos-i-kak-on-rabotaet

🛠 **Потрогать (на контроллере домена):**
```powershell
Get-ADDomain                          # инфо о домене
Get-ADUser -Filter * | Select Name, SamAccountName
Get-ADGroup -Filter * | Select Name
Get-ADComputer -Filter *
Get-ADUser -Filter {ServicePrincipalName -like "*"} -Properties ServicePrincipalName  # юзеры с SPN (цели Kerberoasting!)
dir C:\Windows\NTDS\                   # увидишь NTDS.dit
```
Задание: создай в домене сервисную учётку с SPN, объясни, почему она становится мишенью.

💬 **Промпт мне:** «Объясни иерархию AD (forest/tree/domain/OU) на примере, и почему NTDS.dit и krbtgt — самые ценные цели атакующего».

---

## Аутентификация в AD

- [ ] NTLM — challenge-response, NTLMv1/v2
- [ ] Kerberos — основной протокол: Client, KDC (AS + TGS), Service; AS-REQ/REP → TGT; TGS-REQ/REP → service ticket; PAC

📖 **Читать (приоритет — ссылка друзей):** Ardent101 «Kerberos. Теория» https://ardent101.github.io/posts/kerberos_theory/ ; Habr «Kerberos простыми словами» https://habr.com/ru/articles/803163/

🛠 **Потрогать:**
```powershell
klist                          # текущие Kerberos-билеты (TGT и service tickets)
# залогинься на ресурс, потом снова klist — увидишь новый service ticket
klist purge                    # очистить билеты
```
Задание: нарисуй на бумаге поток Kerberos: AS-REQ → AS-REP(TGT) → TGS-REQ → TGS-REP(ST) → AP-REQ. Объясни, что чем шифруется.

💬 **Промпт мне:** «Проведи меня по Kerberos по шагам как по диалогу клиент↔KDC↔сервис. На каждом шаге: что отправляется, чем шифруется, что внутри. Потом задай мне 5 вопросов на проверку».

---

## Где хранятся секреты

- [ ] Локально: SAM, LSA secrets, LSASS-память (см. [[04 - Windows]])
- [ ] Домен: NTDS.dit на DC, cached credentials на хостах
- [ ] DPAPI

📖 **Читать:** Habr «Хранение паролей Windows» https://habr.com/ru/post/114150/

💬 **Промпт мне:** «Составь карту: где в Windows/AD хранятся секреты (SAM, LSASS, NTDS.dit, LSA secrets, DPAPI, cached creds), что в каждом, и какой инструмент это извлекает».

---

## Атаки на AD (атака + детект)

- [ ] **Kerberoasting** (T1558.003) → детект: Event 4769 с RC4 (0x17), всплеск TGS-запросов
- [ ] **AS-REP Roasting** → детект: 4768 без preauth
- [ ] **Pass-the-Hash** (T1550.002)
- [ ] **Pass-the-Ticket** (T1550.003)
- [ ] **Overpass-the-Hash / Pass-the-Key**
- [ ] **Golden Ticket** (T1558.001) — через krbtgt-хеш
- [ ] **Silver Ticket** (T1558.002)
- [ ] **DCSync** (T1003.006) → детект: 4662 репликация не от DC
- [ ] **DCShadow**
- [ ] **NTLM Relay** (Responder + ntlmrelayx)
- [ ] **Unconstrained / Constrained delegation abuse**
- [ ] **BloodHound** — граф путей атаки → детект: массовые LDAP-запросы

📖 **Читать:** Habr (PT) «Погружение в AD: атаки и детект» https://habr.com/ru/companies/pt/articles/423903/ ; Xakep «Разбираем атаки на AD» https://xakep.ru/2018/06/13/ad-attacks/ ; SecurityLab «Атака DCSync» https://www.securitylab.ru/blog/personal/xiaomite-journal/359242.php

🛠 **Потрогать (с Kali, impacket; домен — твоя ВМ):**
```bash
# Kerberoasting — запросить ST для аккаунтов с SPN:
impacket-GetUserSPNs corp.local/user:password -dc-ip <DC_IP> -request

# AS-REP Roasting:
impacket-GetNPUsers corp.local/ -usersfile users.txt -dc-ip <DC_IP>

# DCSync (если есть права репликации):
impacket-secretsdump corp.local/admin:password@<DC_IP>

# BloodHound сбор данных:
bloodhound-python -u user -p password -d corp.local -ns <DC_IP> -c all
```
**ДЕТЕКТ (на DC, главное для SOC):**
```powershell
# Kerberoasting — найти 4769 с RC4:
Get-WinEvent -LogName Security | Where-Object {$_.Id -eq 4769} |
  Where-Object {$_.Message -like "*0x17*"}   # 0x17 = RC4, подозрительно
# DCSync — найти 4662 с правами репликации не от DC
```
Задание: проведи Kerberoasting со своей Kali против лаб-домена, потом найди событие 4769 с RC4 на контроллере. Связка атака→детект — твоя главная компетенция.

💬 **Промпт мне:** «Для каждой атаки AD (Kerberoasting, AS-REP, PtH, PtT, Golden/Silver Ticket, DCSync) дай: суть в 2 предложениях, инструмент, EventID для детекта, что искать в полях. Таблицей».

---

## Инструменты (знать что делают)

- [ ] mimikatz, Rubeus (Kerberos), impacket (secretsdump, ntlmrelayx, GetUserSPNs)
- [ ] BloodHound / SharpHound, PowerView, CrackMapExec / NetExec, Responder

📖 **Читать:** codeby «10 техник атак на AD» https://codeby.net/threads/10-metodov-atak-na-active-directory-uglublennyi-razbor-i-zashchita.85281/

💬 **Промпт мне:** «Дай шпаргалку: инструмент → что делает → на каком этапе атаки на AD применяется → что оставляет в логах для детекта».

---

## Progress

- [ ] Раздел AD пройден полностью
- [ ] Лаб-домен поднят, проведена хотя бы одна атака с детектом
- [ ] Нарисован поток Kerberos по памяти

---

## 🔭 Связь со следующими шагами

- **→ Инфрапентест (твоя цель):** этот раздел — половина внутреннего пентеста. Дальше: HTB Pro Labs (Dante, Offshore), курс CRTP/CRTE, «Pentest AD 2026» гайды.
- **→ Blueteam:** детект AD-атак (4769, 4662, 4624 корреляция) — топовая компетенция SOC. Дальше: BloodHound для защиты, Microsoft Defender for Identity, Sigma-правила для Kerberos.

---

## 🧪 Тест для повторения

> [!question]- 1. Опиши работу Kerberos по шагам.
> Клиент → AS-REQ к KDC, получает TGT (AS-REP, зашифрован ключом krbtgt). С TGT → TGS-REQ за service ticket, получает TGS-REP. Предъявляет ST сервису (AP-REQ). PAC внутри тикета несёт данные авторизации (группы).

> [!question]- 2. Kerberoasting: суть и детект.
> Запрос service ticket для аккаунта с SPN; ST зашифрован хешем пароля сервисной учётки → offline-брут. Детект: Event 4769 с типом шифрования RC4 (0x17), всплеск TGS-запросов от одного пользователя к множеству SPN.

> [!question]- 3. Golden Ticket vs Silver Ticket.
> Golden — поддельный TGT с хешем krbtgt → доступ ко всему домену, имперсонация любого. Silver — поддельный service ticket с хешем сервисной учётки → доступ только к этому сервису, но без обращения к KDC (тише, нет 4769 на DC).

> [!question]- 4. Что такое DCSync и как детектить?
> Имитация контроллера домена: запрос репликации (DRSUAPI GetNCChanges) для выгрузки хешей включая krbtgt. Детект: Event 4662 с правом DS-Replication-Get-Changes от хоста, не являющегося DC; на сети — DRSUAPI от не-DC.

> [!question]- 5. Pass-the-Hash vs Pass-the-Ticket.
> PtH — аутентификация по NTLM-хешу без пароля (NTLM). PtT — использование украденного Kerberos-тикета (TGT/TGS). Разные протоколы, оба обходят необходимость пароля.

> [!question]- 6. Где хранятся все хеши домена и чем опасен krbtgt?
> Все хеши — в NTDS.dit на DC. krbtgt — учётка, чьим хешем подписываются все TGT; компрометация = возможность ковать Golden Ticket и полный контроль домена.

> [!question]- 7. Что показывает BloodHound и как детектить его сбор?
> Граф отношений AD (юзеры, группы, права, сессии) для поиска путей к Domain Admin. Детект: массовые LDAP-запросы за короткое время, сбор сессий/групп с одного хоста.

> [!question]- 8. Что такое AS-REP Roasting и условие атаки?
> Атака на юзеров с отключённой preauth (Do not require Kerberos preauthentication): можно запросить AS-REP и брутить offline. Детект: Event 4768 без preauth-флага.

> [!question]- 9. Что такое SPN и почему важен для атак?
> Service Principal Name — идентификатор сервиса в AD, привязанный к учётке. Аккаунты с SPN — мишени Kerberoasting (можно запросить их ST). Поиск: `setspn -Q */*` или PowerShell-фильтр по ServicePrincipalName.

> [!question]- 10. Что такое NTLM Relay и чем опасен?
> Атакующий перехватывает NTLM-аутентификацию и релеит её на другой сервис (ntlmrelayx), аутентифицируясь как жертва без знания пароля/хеша. Защита: SMB signing, EPA. Инструменты: Responder + ntlmrelayx.

> [!question]- 11. Что такое Overpass-the-Hash (Pass-the-Key)?
> Использование NT-хеша для получения легитимного Kerberos TGT (вместо прямого NTLM). Позволяет «превратить» хеш в Kerberos-билеты, обходя детекты на чистый PtH.

> [!question]- 12. Что такое unconstrained delegation и риск?
> Режим, при котором сервис может имперсонировать пользователя к любому ресурсу. Если атакующий компрометирует такой хост, он может собрать TGT любого, кто к нему подключился (включая DC через coercion). Высокий риск.

> [!question]- 13. Какие EventID — главные для детекта AD-атак?
> 4768 (TGT — AS-REP roasting), 4769 (ST — Kerberoasting, смотреть RC4 0x17), 4662 (DCSync), 4624/4625 (логоны для корреляции PtH/PtT), 4720/4728 (создание учёток/добавление в группы).

> [!question]- 14. Что такое PAC в Kerberos?
> Privilege Attribute Certificate — структура внутри тикета с данными об авторизации пользователя (SID, членство в группах). Подписывается krbtgt. Подделка PAC — основа Golden Ticket.

---

## 📚 Все источники раздела (сводно)

**Теория Kerberos**
- Ardent101 (друзья) — теория: https://ardent101.github.io/posts/kerberos_theory/
- Habr — Kerberos простыми словами: https://habr.com/ru/articles/803163/
- merionet — что такое Kerberos: https://wiki.merionet.ru/articles/cto-takoe-kerberos-i-kak-on-rabotaet

**Атаки + детект**
- Habr (PT) — Погружение в AD: https://habr.com/ru/companies/pt/articles/423903/
- Xakep — атаки на AD: https://xakep.ru/2018/06/13/ad-attacks/
- SecurityLab — DCSync: https://www.securitylab.ru/blog/personal/xiaomite-journal/359242.php
- codeby — 10 техник атак на AD: https://codeby.net/threads/10-metodov-atak-na-active-directory-uglublennyi-razbor-i-zashchita.85281/

**Инструменты**
- mimikatz wiki: https://github.com/gentilkiwi/mimikatz/wiki
- impacket: https://github.com/fortra/impacket
- BloodHound: https://github.com/SpecterOps/BloodHound
- Rubeus: https://github.com/GhostPack/Rubeus

**Практика**
- TryHackMe — Active Directory path: https://tryhackme.com/module/hacking-active-directory
