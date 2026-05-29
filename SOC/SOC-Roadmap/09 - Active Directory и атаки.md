---
tags: [soc, activedirectory, kerberos, ntlm, roadmap]
section: 09
---

# 09 - Active Directory и атаки

[[00 - SOC Junior Roadmap (BiZone)]]

> Самый частый вектор в корпоративе. Высокий приоритет.

## Основы AD

- [ ] Domain, Forest, Tree, OU, Domain Controller
- [ ] Объекты: User, Computer, Group, GPO
- [ ] LDAP — протокол доступа к каталогу
- [ ] SYSVOL, NTDS.dit (`C:\Windows\NTDS\NTDS.dit` — база AD со всеми хэшами)
- [ ] krbtgt-аккаунт — ключ ко всему домену
- [ ] SPN (Service Principal Name)

## Аутентификация в AD

- [ ] NTLM — challenge-response, где используется, NTLMv1/v2
- [ ] Kerberos — основной протокол
  - [ ] Участники: Client, KDC (AS + TGS), Service
  - [ ] AS-REQ/AS-REP → TGT
  - [ ] TGS-REQ/TGS-REP → service ticket
  - [ ] PAC (Privilege Attribute Certificate)
  - [ ] Изучить теорию: ardent101.github.io/posts/kerberos_theory

## Где хранятся секреты

- [ ] Локально: SAM, LSA secrets, LSASS-память (см. [[04 - Windows]])
- [ ] Домен: NTDS.dit на DC, cached credentials на хостах
- [ ] DPAPI

## Атаки на AD (атака + детект)

- [ ] **Kerberoasting** (T1558.003) — запрос service ticket, offline-брут
  - [ ] Детект: Event 4769 с RC4 (encryption type 0x17), всплеск запросов
- [ ] **AS-REP Roasting** — пользователи без preauth
  - [ ] Детект: Event 4768 без preauth
- [ ] **Pass-the-Hash** (T1550.002) — NTLM-хэш вместо пароля
- [ ] **Pass-the-Ticket** (T1550.003) — кража Kerberos-тикета
- [ ] **Overpass-the-Hash / Pass-the-Key**
- [ ] **Golden Ticket** (T1558.001) — поддельный TGT (через krbtgt-хэш)
- [ ] **Silver Ticket** (T1558.002) — поддельный service ticket
- [ ] **DCSync** (T1003.006) — имитация репликации, дамп хэшей
  - [ ] Детект: Event 4662 репликация не от DC
- [ ] **DCShadow** — поддельный DC
- [ ] **NTLM Relay** (Responder + ntlmrelayx)
- [ ] **Unconstrained / Constrained delegation abuse**
- [ ] **BloodHound** — разведка путей атаки (граф)
  - [ ] Детект: массовые LDAP-запросы

## Инструменты (знать что делают)

- [ ] mimikatz, Rubeus (Kerberos), impacket (secretsdump, ntlmrelayx, GetUserSPNs)
- [ ] BloodHound / SharpHound, PowerView, CrackMapExec / NetExec, Responder

## Progress

- [ ] Раздел Active Directory пройден полностью

---

## 🧪 Тест для повторения

> [!question]- 1. Опиши работу Kerberos по шагам.
> Клиент → AS-REQ к KDC, получает TGT (AS-REP), зашифрованный ключом krbtgt. С TGT → TGS-REQ за service ticket к нужному сервису, получает TGS-REP. Предъявляет service ticket сервису. PAC внутри тикета несёт данные об авторизации (группы).

> [!question]- 2. Kerberoasting: суть и детект.
> Запрос service ticket (TGS) для аккаунта с SPN; тикет зашифрован хэшем пароля сервисной учётки → offline-брут. Детект: Event 4769 с типом шифрования RC4 (0x17), всплеск запросов TGS от одного пользователя к множеству SPN.

> [!question]- 3. Чем Golden Ticket отличается от Silver Ticket?
> Golden — поддельный TGT, созданный с хэшем krbtgt → доступ ко всему домену, имперсонация любого. Silver — поддельный service ticket с хэшем конкретной сервисной учётки → доступ только к этому сервису, но без обращения к KDC (тише).

> [!question]- 4. Что такое DCSync и как детектить?
> Имитация контроллера домена: запрос репликации каталога (DRSUAPI GetNCChanges) для выгрузки хэшей, включая krbtgt. Инструмент: mimikatz `lsadump::dcsync`, secretsdump. Детект: Event 4662 с правами репликации (DS-Replication-Get-Changes) от хоста, который не является DC.

> [!question]- 5. Pass-the-Hash vs Pass-the-Ticket.
> PtH — аутентификация по NTLM-хэшу без знания пароля (NTLM). PtT — использование украденного Kerberos-тикета (TGT/TGS). Оба обходят необходимость пароля; разные протоколы аутентификации.

> [!question]- 6. Где в AD хранятся все хэши домена и чем опасен krbtgt?
> Все хэши — в NTDS.dit на контроллере домена (`C:\Windows\NTDS\NTDS.dit`). krbtgt — учётка, чьим хэшем подписываются все TGT; компрометация = возможность ковать Golden Ticket и тотальный контроль домена.

> [!question]- 7. Что показывает BloodHound и как его активность детектить?
> Строит граф отношений в AD (пользователи, группы, права, сессии) для поиска кратчайших путей к Domain Admin. Детект: массовые LDAP-запросы за короткое время, сбор данных о сессиях/группах с одного хоста.

---

## 📚 Источники для подготовки

**Теория Kerberos (приоритет — ссылка от друзей)**
- Ardent101 — Kerberos для специалиста по пентесту, ч.1 Теория (AS-REQ/REP, TGT, ST, PAC): https://ardent101.github.io/posts/kerberos_theory/
- Habr — Kerberos простыми словами (флаги, делегирование): https://habr.com/ru/articles/803163/
- merionet wiki — Что такое Kerberos и как он работает (KDC, AS, TGS, ключи): https://wiki.merionet.ru/articles/cto-takoe-kerberos-i-kak-on-rabotaet

**Атаки на AD + детект (главное для SOC)**
- Habr (PT) — Погружение в AD: продвинутые атаки и способы детекта (PtH, Overpass-the-Hash, Kerberos): https://habr.com/ru/companies/pt/articles/423903/
- Xakep — Разбираем атаки на Active Directory: техники проникновения и детекта (7 заклинаний): https://xakep.ru/2018/06/13/ad-attacks/
- SecurityLab — Атака DCSync: кража хэшей через репликацию + реагирование: https://www.securitylab.ru/blog/personal/xiaomite-journal/359242.php

**Инструменты атакующих (понимать что делают)**
- mimikatz (официальная wiki): https://github.com/gentilkiwi/mimikatz/wiki
- impacket (secretsdump, ntlmrelayx, GetUserSPNs): https://github.com/fortra/impacket
- BloodHound (граф путей атаки в AD): https://github.com/SpecterOps/BloodHound
- Rubeus (Kerberos-атаки): https://github.com/GhostPack/Rubeus

**Детект-справочник**
- Ultimate Windows Security — EventID 4769 (Kerberoasting), 4662 (DCSync), 4768 (AS-REP): https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/
