# SOC L1 — Темы не раскрытые в БД (BI.ZONE вакансия)
# Формат: Вопрос | Ответ (разделитель |)
# Покрывает: AD основы / Сети TCP/IP / Криптография / Windows безопасность / Linux основы / SIEM платформы / EDR / PowerShell / TI / CVE/CVSS / ИБ основы

---

## ══ ИБ ОСНОВЫ (CIA, AAA, Zero Trust) ══

CIA Triad — что это и примеры нарушений|Три столпа ИБ. Confidentiality (конфиденциальность): доступ только авторизованным — нарушение: утечка данных, перехват трафика. Integrity (целостность): данные не изменены без авторизации — нарушение: tampering, man-in-the-middle изменяет данные. Availability (доступность): система работает когда нужно — нарушение: DDoS, ransomware шифрует серверы.

AAA — что такое и примеры систем|Authentication (аутентификация): подтверждение личности — кто ты? (пароль, биометрия, токен). Authorization (авторизация): что тебе разрешено? (RBAC, ACL). Accounting (учёт): что ты делал? (логи, аудит). Примеры систем AAA: RADIUS, TACACS+, Kerberos. В SOC: Authentication failure = 4625, Authorization = 4663, Accounting = все логи.

Аутентификация: факторы и типы|Фактор 1 — Знание: пароль, PIN, секретный вопрос. Фактор 2 — Владение: токен, смарт-карта, TOTP приложение, SMS. Фактор 3 — Биометрия: отпечаток, лицо, сетчатка. MFA (Multi-Factor): 2+ факторов разных типов. SSO (Single Sign-On): одна аутентификация = доступ ко всем системам. Federated Identity: доверие между организациями (SAML, OAuth).

Принцип наименьших привилегий (PoLP)|Пользователь/процесс получает только минимально необходимые права для выполнения задачи. Примеры: сервисный аккаунт только с правами на чтение БД (не DA), разработчик без прав на prod. Нарушение: широкие AD права у всех, Everyone: Full Control на шарах. SOC: аудит чрезмерных привилегий — AD пользователи в группе DA кто не должен быть.

Defense in Depth (эшелонированная оборона)|Несколько уровней защиты — если один обойдён, следующий сдерживает. Уровни: периметр (firewall/IPS), сеть (NDR, VLAN сегментация), хост (EDR, hardening), приложение (WAF, код), данные (шифрование, DLP), пользователь (обучение, MFA). Концепция: атакующий должен обойти ВСЕ уровни.

Типы контролей безопасности|Preventive (превентивные): предотвращают — firewall, AV, шифрование, политики паролей. Detective (детектирующие): обнаруживают — SIEM, IDS, audit logs, Honeypot. Corrective (корректирующие): устраняют после инцидента — patch, restore backup, изоляция хоста. Deterrent (сдерживающие): отпугивают — предупреждения, юридические меры. Compensating: замещают основной контроль.

Zero Trust — принципы|"Никогда не доверяй, всегда проверяй". Принципы: 1. Verify Explicitly — аутентифицируй каждый запрос (MFA, контекст). 2. Least Privilege Access — минимальные права, JIT (just-in-time). 3. Assume Breach — проектируй с допущением что атакующий уже внутри. Применение: micro-segmentation, mTLS между сервисами, постоянный мониторинг. Противоположность: доверие по периметру (устарело).

Безопасность по умолчанию (Secure by Default)|Продукты поставляются с безопасными настройками, не нужно дополнительно защищать. Примеры: пароли на устройствах уникальны из коробки, шифрование включено по умолчанию, ненужные порты закрыты. Антипаттерн: Telnet вместо SSH, пустой пароль admin, открытый MongoDB 27017 на 0.0.0.0.

Threat Modelling — что такое STRIDE|Метод анализа угроз. S=Spoofing (подмена идентичности), T=Tampering (изменение данных), R=Repudiation (отрицание действий), I=Information Disclosure (раскрытие), D=Denial of Service (недоступность), E=Elevation of Privilege (повышение прав). Для каждой функции системы — какие STRIDE угрозы применимы?

---

## ══ ACTIVE DIRECTORY — ОСНОВЫ ══

Active Directory: что такое и зачем|Служба каталогов Microsoft для централизованного управления объектами в домене (пользователи, компьютеры, группы, политики). Основа корпоративной инфраструктуры. Функции: единая точка аутентификации (Kerberos), централизованное управление политиками (GPO), организация иерархии (OU), делегирование администрирования.

AD: структура — Forest, Domain, OU|Forest (лес): верхний уровень, содержит домены с общей схемой. Domain (домен): административная граница, общая БД (NTDS.dit). Organizational Unit (OU): контейнер внутри домена для организации объектов, к ОУ привязываются GPO. Site: географическая привязка (для оптимизации репликации). Отношение: один лес = много доменов, один домен = много ОУ.

AD: типы групп (Security vs Distribution, Group Scope)|Security Group: для назначения прав (ACL, GPO). Distribution Group: только для рассылки email (в AD, Exchange). По охвату: Local (только в домене где создана), Global (члены только из своего домена), Universal (члены из любого домена леса, репликация в GC). Правило вложения: A G DL (Account→Global→Domain Local для назначения прав).

AD: что такое GPO и как работает|Group Policy Object — объект политики группы. Содержит настройки конфигурации (реестр, файловая система, сеть, безопасность, скрипты). Привязывается к Site/Domain/OU. Применяется при загрузке компьютера (Computer Configuration) и при входе пользователя (User Configuration). Порядок применения: Local → Site → Domain → OU (LSDOU), последний выигрывает. gpupdate /force — принудительное обновление.

AD: SID — что такое|Security Identifier — уникальный идентификатор объекта (пользователя, группы, компьютера) в домене. Формат: S-1-5-21-<домен>-<RID>. Не меняется при переименовании объекта. Используется в ACL вместо имени. Well-known SIDs: S-1-1-0 = Everyone, S-1-5-18 = SYSTEM, S-1-5-32-544 = Administrators. RID 500 = Administrator, RID 501 = Guest, RID 502 = krbtgt.

AD: FSMO роли — зачем|Flexible Single Master Operations — 5 ролей, не допускающих конфликтов: Schema Master (изменение схемы AD, 1 на лес), Domain Naming Master (добавление/удаление доменов, 1 на лес), PDC Emulator (синхронизация времени, блокировки, репликация паролей, 1 на домен), RID Master (выдача RID пулов DC, 1 на домен), Infrastructure Master (межд. объекты, 1 на домен). Узнать: netdom query fsmo.

AD: LDAP — что такое и синтаксис запросов|Lightweight Directory Access Protocol — протокол доступа к AD (порт 389, LDAPS: 636). Дерево объектов: DN (Distinguished Name) = полный путь: CN=John,OU=IT,DC=corp,DC=local. Базовые операции: BIND (аутентификация), SEARCH (поиск объектов), ADD/MODIFY/DELETE. Фильтры: (&(objectClass=user)(memberOf=CN=Admins,...)) — все пользователи в группе. Атакующие: SharpHound/BloodHound делает LDAP dump.

AD: репликация и Global Catalog|Все DC в домене реплицируют NTDS.dit друг другу (multi-master). Global Catalog (GC): частичная копия всех объектов леса (порт 3268/3269). Нужен для входа (проверка Universal Groups). Если GC недоступен — вход замедляется. Репликация: USN (Update Sequence Number), уведомления об изменениях, конфликт: последняя запись побеждает. Знать: KCC (Knowledge Consistency Checker) строит топологию репликации.

AD: доверительные отношения (Trust)|Trust позволяет пользователям одного домена/леса аутентифицироваться в другом. Типы: One-way (одностороннее: A доверяет B, B не доверяет A), Two-way (двустороннее). Transitive: доверие передаётся (A доверяет B, B доверяет C → A доверяет C). Forest Trust: между лесами. External Trust: к конкретному домену другого леса. Атакующие: trust abuse для движения между доменами.

AD: ACL/DACL/SACL — что это|ACL (Access Control List): список контроля доступа на объекте AD. DACL (Discretionary ACL): кто имеет какой доступ — список ACE (Access Control Entry). SACL (System ACL): аудит — какие операции логировать. ACE = SID + права. Опасные права AD: GenericAll, GenericWrite, WriteDACL, WriteOwner, DCSync-права (DS-Replication-Get-Changes). Детект изменений ACL: EID 5136 + 4662.

AD: Organizational Units vs Группы — разница|OU (Organizational Unit): контейнер для организации объектов И применения GPO. Не используются в ACL для прав доступа. Группы: для назначения прав (Security Groups используются в ACL). Частая ошибка: давать доступ по OU вместо группы = неверно. Делегирование прав: можно делегировать управление OU определённым администраторам.

AD: что проверять при первичном расследовании|Подозрительная учётка: 1. Last Logon (когда последний раз входила?). 2. Member Of (в каких группах?). 3. Password Last Set (давно не менялся = забытая учётка). 4. Enabled/Disabled. 5. Description (часто пароль написан там!). 6. AdminCount=1 (защищённая учётка, была в привилегированной группе). Команды: Get-ADUser -Identity username -Properties *.

---

## ══ СЕТИ — TCP/IP ПРОТОКОЛЫ ДЕТАЛЬНО ══

IP адресация: классы, CIDR, маски|Классы (устарели): A (0.x, /8), B (128-191.x, /16), C (192-223.x, /24). Private ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 — не маршрутизируются в интернете. CIDR: /24 = 256 адресов (254 хоста), /25 = 128, /16 = 65536. Loopback: 127.0.0.0/8. Link-local: 169.254.0.0/16 (APIPA — нет DHCP ответа). SOC: знать private vs public для атрибуции.

ARP протокол детально|Address Resolution Protocol — L2. Резолвит IP → MAC в пределах сети. ARP Request: broadcast "Кто имеет IP X, скажи мне MAC". ARP Reply: unicast "IP X = MAC YY:YY:...". ARP кеш: arp -a (Windows), ip neigh (Linux). Gratuitous ARP: устройство анонсирует свой IP→MAC без запроса. Уязвимость: нет аутентификации — любой может ответить на ARP Request (ARP Spoofing). ARP только внутри broadcast domain (VLAN).

ICMP — типы и коды важные для SOC|Internet Control Message Protocol — служебные сообщения L3. Тип 0 Echo Reply (ответ на ping). Тип 8 Echo Request (ping). Тип 3 Destination Unreachable (Code 0=network, 1=host, 3=port, 13=filtered by firewall). Тип 11 Time Exceeded (TTL=0, traceroute использует). Тип 5 Redirect (атака перенаправления маршрутов). ICMP tunneling: данные в payload Echo Request/Reply.

TCP vs UDP — сравнительная таблица|TCP: надёжный, установление соединения (3-way handshake), управление потоком, упорядоченная доставка, ack подтверждения. Примеры: HTTP, HTTPS, SSH, FTP, SMTP. UDP: ненадёжный, без соединения, низкая задержка, нет гарантии доставки. Примеры: DNS, DHCP, NTP, SNMP, TFTP, VoIP. SOC: UDP scan быстрее но ненадёжен, TCP SYN scan — стандарт nmap.

HTTP методы и статус коды|Методы: GET (получить), POST (отправить данные), PUT (создать/заменить), PATCH (обновить), DELETE (удалить), HEAD (только заголовки), OPTIONS (возможные методы). Статусы: 2xx успех (200 OK, 201 Created), 3xx перенаправление (301 постоянное, 302 временное), 4xx ошибка клиента (400 Bad Request, 401 Unauth, 403 Forbidden, 404 Not Found, 429 Too Many Requests), 5xx ошибка сервера (500 Internal, 503 Unavailable). SOC: 4xx спам = сканирование, 200 на POST к *.php = webshell.

HTTP заголовки важные для SOC|Запрос: Host (целевой домен), User-Agent (браузер/инструмент — аномалии: curl, python-requests, sqlmap), Content-Type (что отправляем), Authorization (Bearer token, Basic auth), Cookie. Ответ: Content-Type (что вернули), Set-Cookie (установка cookie), X-Powered-By (раскрывает технологию = fingerprinting), Server (версия — info leakage), Location (редирект). Детект: нестандартный User-Agent + POST = сканирование/атака.

TLS handshake — пошагово|1. Client Hello: версия TLS, cipher suites, случайный bytes, SNI (имя сайта). 2. Server Hello: выбранный cipher suite, случайный bytes. 3. Certificate: сервер отправляет X.509 сертификат. 4. Client: проверяет сертификат (CA chain, срок, CN/SAN). 5. Key Exchange: согласование сессионного ключа (ECDHE — perfect forward secrecy). 6. Finished: оба стороны подтверждают шифрование. JA3 снимается с Client Hello.

TLS: сертификат X.509 — поля|Subject (кому выдан: CN=*.example.com), Issuer (кто выдал: DigiCert CA), Validity (срок: notBefore, notAfter), SAN (Subject Alternative Names: список доменов), Public Key (RSA 2048+ или ECDSA), Signature (подпись CA). Проверка браузером: 1. CA в trusted store? 2. Не истёк? 3. CN/SAN совпадает с доменом? 4. Не отозван (CRL/OCSP)? Self-signed = не в trusted store = предупреждение.

Firewall типы: stateless vs stateful vs NGFW|Stateless (пакетный фильтр): проверяет каждый пакет по правилам (src IP, dst IP, порт, протокол) без учёта контекста соединения. Stateful: отслеживает состояния соединений (connection table) — разрешает ответный трафик автоматически. NGFW (Next-Gen): DPI (Deep Packet Inspection), IPS, App ID, User ID, SSL inspection, threat intelligence. WAF: специализирован под веб (L7), знает HTTP.

Proxy vs Reverse Proxy|Forward Proxy: клиент → Proxy → Интернет. Скрывает клиента, кеширует, контентная фильтрация, логирует запросы. Атакующие используют для анонимизации. Reverse Proxy: Интернет → Proxy → Backend сервер. Скрывает серверы, балансировка нагрузки, SSL termination, WAF. Примеры: nginx как reverse proxy. SOC: proxy логи = видимость всего HTTP(S) трафика пользователей (если есть SSL inspection).

Маршрутизация: основные концепции|Default Gateway: IP роутера — куда отправлять пакеты не для локальной сети. Routing Table: таблица маршрутов (ip route / netstat -r / route print). Метрика: приоритет маршрута. Static Route: вручную прописанный маршрут. Dynamic Routing: протоколы (OSPF внутри организации, BGP между AS в интернете). TTL: Time-to-Live — декрементируется на каждом hop (traceroute использует для карты маршрута).

Что такое VLAN и зачем|Virtual LAN — логическая сегментация сети на L2 (без физического разделения). Цели: изоляция трафика (пользователи от серверов, IoT от корпоративных ПК), безопасность, управление broadcast доменами. Trunk port: несёт трафик нескольких VLAN (802.1Q тегирование). Access port: один VLAN. Native VLAN: нетегированный трафик на trunk. SOC: трафик между VLAN проходит через роутер/L3 коммутатор = логируется.

---

## ══ КРИПТОГРАФИЯ — ОСНОВЫ ══

Симметричное vs асимметричное шифрование|Симметричное: один ключ для шифрования и дешифрования. Быстро. Алгоритмы: AES (128/192/256 бит), 3DES (устарел), ChaCha20. Проблема: как безопасно передать ключ? Асимметричное: пара ключей (публичный + приватный). Публичный шифрует / приватный дешифрует (или наоборот для подписи). Медленнее. Алгоритмы: RSA (2048+ бит), ECC (P-256). На практике: асимметричное для обмена ключом + симметричное для данных (TLS).

Хеширование — свойства и алгоритмы|Хеш-функция: произвольные данные → фиксированный дайджест фиксированного размера. Свойства: детерминированность (один вход = один хеш), лавинный эффект (1 бит изменён = другой хеш), необратимость (нельзя получить данные из хеша), устойчивость к коллизиям. MD5: 128 бит, BROKEN (коллизии). SHA-1: 160 бит, BROKEN. SHA-256: 256 бит, безопасен. SHA-3/Keccak: современный стандарт. NTLM использует MD4.

Хеш vs Шифрование — разница|Хеш: одностороннее преобразование — нельзя получить исходные данные обратно. Используется для: проверки целостности, хранения паролей, цифровые подписи. Шифрование: двустороннее — с ключом можно расшифровать. Частая ошибка: хранить пароли в зашифрованном виде (если ключ скомпрометирован — все пароли открыты). Правильно: хеш + соль. SOC: пароли в NTDS.dit хранятся как NTLM хеши (MD4).

Соль (Salt) в хешировании|Случайные данные добавляемые к паролю ПЕРЕД хешированием. Хранится вместе с хешем (открыто). Цель: уникальный хеш для одного пароля у разных пользователей → невозможно атаковать rainbow tables и precomputed hashes. Bcrypt, Argon2, PBKDF2 — медленные хеш-функции с солью для паролей (slow by design). NTLM хеши НЕ солёные → rainbow tables работают.

Цифровая подпись — как работает|1. Отправитель: вычисляет хеш сообщения → шифрует хеш своим ПРИВАТНЫМ ключом = подпись. 2. Получатель: расшифровывает подпись ПУБЛИЧНЫМ ключом отправителя = получает хеш → сравнивает с хешем сообщения. Если совпадает: сообщение не изменено И отправлено тем кто имеет приватный ключ (аутентичность + целостность). DKIM использует этот принцип.

PKI — что такое и компоненты|Public Key Infrastructure — инфраструктура управления публичными ключами и сертификатами. Компоненты: CA (Certificate Authority) — выдаёт и подписывает сертификаты (Root CA → Intermediate CA → End-entity cert). RA (Registration Authority) — верифицирует личность заявителя. CRL (Certificate Revocation List) — список отозванных сертификатов. OCSP — онлайн проверка отзыва. Цепочка доверия: браузер доверяет Root CA → Intermediate → ваш сертификат.

Типы сертификатов и проверка CA|DV (Domain Validation): только проверка владения доменом (быстро, автоматически, Let's Encrypt). OV (Organization Validation): проверка организации (ручная). EV (Extended Validation): расширенная проверка (зелёный замок в старых браузерах). Self-signed: подписан сам собой — не trusted. SOC: атакующие используют Let's Encrypt сертификаты для C2 (HTTPS + валидный сертификат ≠ безопасный сайт!).

AES — основные режимы шифрования|ECB (Electronic Codebook): каждый блок шифруется независимо — НЕБЕЗОПАСЕН (паттерны сохраняются). CBC (Cipher Block Chaining): XOR с предыдущим блоком — нужен IV. CTR (Counter): превращает блочный в поточный — параллелизуется. GCM (Galois/Counter Mode): шифрование + аутентификация (AEAD). Используется в TLS 1.3: AES-256-GCM. Ключ: 128/192/256 бит — безопасны все три. Слабое место: слабый IV или переиспользование nonce.

---

## ══ WINDOWS — АРХИТЕКТУРА И МОДЕЛЬ БЕЗОПАСНОСТИ ══

Windows: токены доступа (Access Tokens)|Каждый процесс/поток имеет токен — представление прав пользователя. Содержит: SID пользователя, SIDs групп, привилегии (SeDebugPrivilege, SeImpersonatePrivilege и др.), уровень целостности (Integrity Level). Primary Token: у процесса. Impersonation Token: временный для потока (при impersonation сервис выполняет действия от имени клиента). Elevated Token: после UAC Elevation.

Windows: уровни целостности (Integrity Levels)|Механизм Mandatory Integrity Control. Уровни: Low (Protected Mode браузера, sandbox), Medium (обычные пользователи), High (Administrator, UAC elevation), System (SYSTEM процессы, lsass). Правило: процесс не может записывать в объекты БОЛЕЕ высокого уровня. Помогает: ограничивает что может сделать скомпрометированный браузер. Mimikatz для дампа lsass требует High или System.

Windows: структура реестра — ульи|HKLM (HKEY_LOCAL_MACHINE): общесистемные параметры. HKCU (HKEY_CURRENT_USER): параметры текущего пользователя. HKCR (Classes Root): регистрация файловых типов и COM. HKU (Users): профили всех пользователей. HKCC (Current Config): конфигурация оборудования. Физические файлы (ульи): SYSTEM, SOFTWARE, SAM, SECURITY, NTUSER.DAT. SAM хранит хеши локальных пользователей.

Windows реестр: ключевые пути для SOC|Persistence Run Keys: HKCU\Software\Microsoft\Windows\CurrentVersion\Run (пользовательский), HKLM\Software\Microsoft\Windows\CurrentVersion\Run (системный). Services: HKLM\SYSTEM\CurrentControlSet\Services\. COM Hijacking: HKCU\Software\Classes\CLSID\. Winlogon: HKLM\...\Winlogon — Userinit, Shell (persistence здесь). Image File Execution Options: HKLM\...\Image File Execution Options — debugger hijacking.

Windows: UAC — как работает|User Account Control. Два токена при входе admin: Filtered Token (medium integrity, для обычной работы) + Full Token (high integrity, по запросу). При запросе elevation: UAC диалог → пользователь подтверждает → процесс получает Full Token. Auto-elevation: некоторые Microsoft процессы поднимаются без диалога (fodhelper, mmc). UAC не защита от малвари с admin правами — только от случайных изменений. Уровни: Always Notify / Default / Never.

Windows: SAM vs NTDS.dit|SAM (Security Account Manager): локальная БД паролей в реестре (HKLM\SAM). Хранит NTLM хеши локальных пользователей. Зашифрована SysKey (BootKey). NTDS.dit: база данных Active Directory на DC. Хранит хеши ВСЕХ пользователей домена. Зашифрована ключом из SYSTEM hive. Дамп SAM: reg save HKLM\SAM + reg save HKLM\SYSTEM → impacket secretsdump. Дамп NTDS: DCSync или ntdsutil snapshot.

Windows: LSA и Credential Manager|LSA (Local Security Authority): компонент ответственный за аутентификацию. LSA Secrets: HKLM\SECURITY\Policy\Secrets — хранит кеши паролей сервисов, последний пароль входа. Credential Manager: DPAPI-зашифрованное хранилище браузерных/Windows credentials (%AppData%\Microsoft\Credentials). Mimikatz: dpapi::cred — расшифровать Credential Manager. lsass.exe хранит в памяти: NTLM хеши, Kerberos тикеты, иногда plaintext (WDigest).

Windows: файловая система NTFS — артефакты|$MFT (Master File Table): запись о каждом файле. $STANDARD_INFORMATION: MACE timestamps (Modified/Accessed/Created/Entry Modified). $FILE_NAME: второй набор timestamps (труднее подделать). ADS (Alternate Data Streams): скрытый поток данных — file.txt:hidden.exe. Zone.Identifier: ADS с информацией откуда скачан файл (Mark of the Web). USN Journal ($UsnJrnl): лог изменений файловой системы — не удаляется автоматически.

Windows: Prefetch и другие execution артефакты|Prefetch: C:\Windows\Prefetch\*.pf — доказательство запуска программы + первые 8 путей к загруженным файлам + timestamp первого/последнего запуска + счётчик запусков. Enabled по умолчанию на рабочих станциях, ВЫКЛЮЧЕН на серверах (SSD). ShimCache (AppCompatCache): в SYSTEM hive — список запускавшихся программ. AmCache.hve: SHA1 хеш исполняемого файла + путь + время первого запуска. Важно при DFIR.

Windows: Защита системных файлов|Windows File Protection / SFC: защита системных файлов в System32. SFC /scannow — проверка и восстановление. WRP (Windows Resource Protection). WDAC (Windows Defender Application Control): контроль запуска приложений. AppLocker: правила на основе пути/хеша/издателя. PPL (Protected Process Light): защита критичных процессов (lsass в PPL — Mimikatz не работает без bypass). Credential Guard: изоляция lsass в Hyper-V контейнере.

---

## ══ LINUX — ОСНОВЫ ══

Linux: права доступа к файлам (chmod)|Формат: rwxrwxrwx = Owner/Group/Others. r=4 (чтение), w=2 (запись), x=1 (выполнение). Числовая запись: 755 = rwxr-xr-x (owner всё, группа и others: r-x). 644 = rw-r--r-- (owner: чтение/запись, остальные: только чтение). Опасные права: 777 (все могут всё), SUID 4xxx (запуск от владельца). chmod +x file, chmod 755 file, chmod o-w file.

Linux: специальные биты (SUID, SGID, Sticky)|SUID (4000): файл запускается с правами ВЛАДЕЛЬЦА (не запускающего). Пример: /usr/bin/passwd — обычный пользователь меняет пароль от root. Опасно: bash с SUID = мгновенный root. SGID (2000): файл запускается с правами ГРУППЫ; на директорию = новые файлы наследуют группу. Sticky bit (1000): на директории (/tmp) — файл удаляет только его создатель. Детект SUID: find / -perm -4000 -type f.

Linux: /etc/passwd и /etc/shadow|/etc/passwd: строка: username:x:UID:GID:comment:home:shell. UID=0 = root. x = пароль в /etc/shadow. /bin/nologin или /bin/false = нет интерактивного входа. /etc/shadow: username:$type$salt$hash:lastchg:min:max:warn:... Тип хеша: $1$=MD5, $2a$=bcrypt, $5$=SHA-256, $6$=SHA-512. Атакующие: добавить строку в /etc/passwd с UID=0 = backdoor root. Детект: auditd на запись в /etc/passwd.

Linux: процессы — ключевые концепции|Каждый процесс имеет: PID (Process ID), PPID (Parent PID), UID (чей процесс), GID, набор дескрипторов файлов. init (PID=1): первый процесс, предок всех. systemd = современный init. ps aux: все процессы. ps -ef: с PPID. pstree: дерево. /proc/PID/: информация о процессе (cmdline, maps, fd, net). Orphan process: родитель умер (усыновляет init/systemd). Zombie: завершился но родитель не вызвал wait().

Linux: сетевые команды для SOC|ip addr: IP адреса интерфейсов (замена ifconfig). ip route: таблица маршрутов. ip neigh: ARP таблица. ss -tulnp: TCP/UDP порты + процессы (замена netstat). ss -antp | grep ESTABLISHED: активные соединения. netstat -tulnp: то же (устарело). lsof -i :443: процессы на порту 443. tcpdump -i eth0 port 443 -w capture.pcap: захват трафика.

Linux: iptables — основные концепции|Цепочки: INPUT (входящий), OUTPUT (исходящий), FORWARD (транзитный). Правила проверяются по порядку, первое совпадение = действие. Действия: ACCEPT, DROP, REJECT (с ответом), LOG. Таблицы: filter (основная), nat (NAT правила), mangle. Просмотр: iptables -L -n -v. Нов. альтернатива: nftables, UFW (ufw allow 22/tcp). Детект: изменения iptables могут скрыть трафик атакующего.

Linux: /proc — что важно знать|Виртуальная файловая система с информацией о процессах и системе. /proc/PID/cmdline: командная строка процесса. /proc/PID/maps: карта памяти. /proc/PID/net/tcp: сетевые соединения. /proc/PID/fd/: открытые файлы/сокеты. /proc/net/arp: ARP таблица. /proc/sys/: настройки ядра (sysctl). Криминалистика: lsof /proc/PID/fd — файлы удалённые но открытые процессом (malware делает это).

Linux: пакетные менеджеры и логи установки|Debian/Ubuntu: apt install package, dpkg -l (список пакетов), /var/log/apt/history.log (история установки). RHEL/CentOS: yum install / dnf install, rpm -qa (список), /var/log/yum.log или /var/log/dnf.log. SOC: атакующие устанавливают инструменты (nmap, netcat, golang) через пакетный менеджер — видно в логах. Или используют LOLbins чтобы не оставлять следов.

Linux: sudo — конфигурация и опасные правила|/etc/sudoers (редактировать только visudo). Формат: user ALL=(ALL:ALL) ALL = полный sudo. user ALL=(ALL) NOPASSWD: /usr/bin/vim = vim без пароля. Опасные правила: NOPASSWD + любой интерпретатор (vim, python, find, less) = немедленная PE. sudo -l: список доступных команд. Детект: auditd на sudo, /var/log/auth.log: "sudo: user : COMMAND=". GTFOBins.github.io: список PE через sudo.

Linux: важные директории для SOC|/etc/: конфигурационные файлы (passwd, shadow, sudoers, crontab, ssh/). /var/log/: логи. /tmp/ и /var/tmp/: временные файлы (writeable всеми — любимое место malware). /home/$USER/: профили пользователей (.ssh/, .bash_history). /root/: home директория root. /proc/: псевдо-ФС ядра. /dev/: устройства. /bin/ /sbin/ /usr/bin/ /usr/sbin/: системные бинари. Persistence: /etc/init.d/, /etc/systemd/system/, /etc/cron.d/.

---

## ══ SIEM ПЛАТФОРМЫ НА ПРАКТИКЕ ══

QRadar — архитектура и компоненты|IBM QRadar SIEM. Console (QRadar Console): интерфейс управления, правила, поиск, offenses. Event Processor (EP): нормализует и коррелирует события. Flow Processor: анализирует NetFlow/sFlow/J-Flow. QFlow Collector: захватывает сетевой трафик. Data Accumulator: агрегация данных. WinCollect: агент для Windows. Log Source Management: настройка источников. Offenses: инциденты генерируемые корреляционными правилами.

QRadar — основные понятия|Offense: алерт/инцидент созданный правилом (как кейс в SOC). Events: нормализованные логи от источников. Flows: записи о сетевом трафике. Log Sources: источники данных (Windows, Linux, Firewall, AV). Правила (Rules): Custom Rules Engine (CRE) — если [условие] то [действие]. QQL: язык запросов (SELECT * FROM events WHERE EventName='...'). Категории событий: нормализованные теги (Exploit, Recon, Auth).

QRadar — работа с Offense L1|При получении offense: 1. Нажать offense → смотреть Source IP, Destination, правило сработавшее. 2. Source/Destination → Asset информация (что за хост?). 3. Log Sources → откуда пришли события. 4. Supporting Events → все события в рамках offense. 5. Добавить notes с анализом. 6. Назначить статус (Follow Up, Under Investigation). 7. Close (False Positive) или Escalate.

ELK Stack — компоненты|Elasticsearch: поисковый движок и хранилище данных (распределённый, индексы). Logstash: pipeline обработки данных (Input → Filter → Output). Kibana: веб-интерфейс для поиска и визуализации. Beats: лёгкие агенты доставки данных (Filebeat=логи, Winlogbeat=Windows Events, Packetbeat=сеть, Auditbeat=auditd). OpenSearch: open-source fork ElasticSearch (Amazon). Elastic Security = SIEM модуль в Kibana.

ELK — язык запросов KQL и Lucene|KQL (Kibana Query Language): user: "john" AND event.action: "logon". Lucene: event_id:4624 AND logon_type:3. Field:value синтаксис. Wildcards: process.name: "power*". Range: @timestamp:[2024-01-01 TO 2024-01-31]. NOT: NOT event_id:4624. Регулярки: process.name:/.*\.exe/. Важно: Discover tab = поиск событий, Dashboard = визуализация, Alerts = правила детекта. Правила в Elastic: EQL (Event Query Language) или KQL.

Splunk — архитектура и Search|Indexer: индексирует и хранит данные (buckets: hot/warm/cold/frozen). Search Head: выполняет поиск. Forwarder: агент доставки данных (Universal Forwarder — легкий, Heavy Forwarder — с парсингом). SPL (Search Processing Language): index=windows EventCode=4624 LogonType=3 | stats count by src_ip, user | where count>10. Основные команды: search, stats, table, eval, rex, lookup, join, timechart. Apps: ES (Enterprise Security) = SIEM поверх Splunk.

MaxPatrol SIEM — что это|Российская SIEM система от Positive Technologies. Интеграция с экосистемой PT: MaxPatrol 8 (сканер уязвимостей), PT NAD (NTA), PT Sandbox, PT XDR. Особенности: граф атак, встроенные экспертные пакеты (PT Expert Security Center), PT Knowledge Base с правилами детекта. Нормализация через схему PT. Корреляционные правила на PT Query Language. Часто используется в российских компаниях (BI.ZONE работает с ним).

SIEM: как работает нормализация событий|Raw Log → Parsing (извлечь поля) → Normalization (привести к единой схеме) → Enrichment (добавить контекст: geo-IP, asset info, TI) → Correlation (применить правила) → Alert. Разные форматы: Windows EventLog (XML), syslog (RFC 5424), CEF (Common Event Format), LEEF (IBM). Проблема: разные вендоры называют поля по-разному — нормализация унифицирует.

SIEM: корреляционное правило — компоненты|Правило состоит из: 1. Условие (condition): паттерн событий — одно событие или sequence. 2. Временное окно (time window): за последние N минут. 3. Группировка (group by): по src_ip или user. 4. Порог (threshold): если N событий. 5. Действие (action): создать алерт, запустить playbook. Пример: IF EID=4625 COUNT>10 по src_ip за 5мин THEN Brute Force Alert.

SIEM: Log Sources — какие подключать приоритетно|Tier 1 (Critical): DC (AD логи), Endpoints (Windows Security/Sysmon), Email Gateway, VPN/Firewall, Web Proxy, DNS сервер. Tier 2 (Important): Веб-серверы (IIS/nginx), Database logи, EDR/AV, Cloud (AWS CloudTrail). Tier 3 (Nice-to-have): IoT, принтеры, физический доступ (СКУД). Правило: логируй там где данные ценны, не всё подряд (alert fatigue).

---

## ══ EDR — ПРИНЦИПЫ И РЕШЕНИЯ ══

EDR — что такое и принципиальные отличия от AV|Endpoint Detection and Response. AV: сигнатуры → ищет known-bad файлы. EDR: поведенческий анализ + телеметрия процессов + сетевые соединения + реестр + ML. Что EDR видит: дерево процессов, загрузка DLL, сетевые соединения от каждого процесса, изменения реестра, файловые операции. Response capabilities: изоляция хоста, kill процесса, карантин файла, memory dump. SIEM видит логи, EDR видит действия.

Как EDR собирает данные (механизмы)|1. Kernel Driver: перехватывает системные вызовы на уровне ядра (максимальная видимость, сложнее обойти). 2. ETW (Event Tracing for Windows): подписывается на провайдеры событий (Microsoft-Windows-Kernel-Process и др.). 3. WFP (Windows Filtering Platform): перехват сетевых соединений. 4. Userland Hooks: DLL инъекция в процессы для перехвата API (уязвимы к unhooking). Лучшие EDR используют kernel driver.

CrowdStrike Falcon — особенности|Один из лидеров рынка. Облачная архитектура: агент (Sensor) + облако (фиксированный cloud-based анализ). AI/ML движок Threat Graph: анализ поведения в реальном времени. Falcon OverWatch: управляемый сервис threat hunting (команда аналитиков). Detection категории: Malware, Exploits, Indicators, Suspicious Activity. RTR (Real Time Response): удалённый доступ к хосту через EDR консоль. Spotlight: управление уязвимостями.

Microsoft Defender for Endpoint (MDE) — что знать|Встроен в Windows (ранее ATP). Интеграция с Azure Sentinel (SIEM), Entra ID (Azure AD), Intune. KQL запросы через Advanced Hunting (тот же язык что в Azure Sentinel). Тесная интеграция с MDTI (Microsoft Defender Threat Intelligence). Defender AV + EDR в одном. Реакция: automated investigation, isolate device, live response. Бесплатен для Windows E5/M365 E5 лицензий.

Kaspersky EDR — что знать SOC|Kaspersky EDR Expert / Optimum. Интеграция с Kaspersky Anti Targeted Attack (KATA) platform. Особенности: YARA правила, IoC-сканирование, threat hunting запросы. Реакция: карантин файла, изоляция хоста, блокировка хеша/URL. В России: один из самых распространённых EDR (особенно в госсекторе). Работает с MaxPatrol SIEM и PT NAD.

EDR Response Actions — что умеет L1|Isolate Host (изолировать хост): отрезает от сети, только EDR канал остаётся. Kill Process: завершить подозрительный процесс. Quarantine File: переместить файл в карантин (EDR sandbox). Block Hash: заблокировать запуск файла по MD5/SHA256 на всех хостах. Block IP/Domain: сетевая блокировка. Memory Dump: получить дамп памяти процесса. Live Response/RTR: командная строка на хосте через EDR. L1 обычно: Isolate + Kill Process + Block Hash. L2: всё остальное.

EDR vs NDR vs SIEM — когда что используется|SIEM: корреляция событий из многих источников, compliance, dashboards, долгосрочное хранение. EDR: глубокая телеметрия с конечных точек, поведенческий детект, response на хосте. NDR (NTA): анализ сетевого трафика, боковое перемещение в сети, beaconing, аномалии трафика. XDR: объединяет все три (EDR+NDR+Email+Cloud) в единый детект и response. L1 SOC использует все: SIEM для алертов, EDR для контекста, NDR для сетевых деталей.

---

## ══ POWERSHELL ДЛЯ SOC ══

PowerShell: режимы выполнения (Execution Policy)|Политика выполнения скриптов — НЕ защита, только препятствие: Restricted (запрет скриптов), AllSigned (только подписанные), RemoteSigned (локальные — можно, скачанные — должны быть подписаны), Unrestricted (всё). Обход: powershell -ExecutionPolicy Bypass -File script.ps1 или powershell -ep bypass. Детект: EID 4104, PowerShell.exe с -ep bypass или -ExecutionPolicy Bypass = suspicious.

PowerShell: Script Block Logging — как включить|EID 4104: логирует ДЕКОДИРОВАННЫЙ блок кода PowerShell (даже если -EncodedCommand). GPO: Computer → Admin Templates → Windows Components → Windows PowerShell → Turn on Script Block Logging. EID 4103: Module Logging (параметры каждой функции). EID 4100: Pipeline execution details. AMSI: отправляет код антивирусу до выполнения. Совет: без EID 4104 — PowerShell чёрный ящик.

PowerShell: ключевые командлеты для IR|Get-Process: список процессов. Get-Process -Name "notepad" | Select-Object Id,Name,Path,Company. Get-NetTCPConnection -State Established: активные TCP соединения. Get-NetTCPConnection | Select LocalAddress,LocalPort,RemoteAddress,RemotePort,OwningProcess. Get-ScheduledTask: все задачи. Get-Service: службы. Get-WinEvent -LogName Security -FilterXPath '*/System/EventID=4624': события по EID. Invoke-Command -ComputerName HOST -ScriptBlock {...}: удалённое выполнение (WinRM).

PowerShell: работа с AD|Get-ADUser -Filter * -Properties *: все пользователи + все атрибуты. Get-ADUser -Identity username -Properties LastLogonDate,PasswordLastSet,MemberOf. Get-ADGroupMember "Domain Admins": члены группы DA. Get-ADComputer -Filter * -Properties LastLogonDate: все компьютеры. Search-ADAccount -LockedOut: заблокированные учётки. Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true}: AS-REP Roastable. Требует: ActiveDirectory модуль (RSAT или AD PowerShell).

PowerShell: Constrained Language Mode (CLM)|Ограниченный языковой режим — блокирует опасные возможности PS: нет Add-Type (нельзя загрузить .NET), нет Invoke-Expression с произвольным кодом, нет COM объектов. Включается WDAC или AppLocker. Проверить: $ExecutionContext.SessionState.LanguageMode → Constrained или FullLanguage. Обходы: использовать старый PowerShell 2 (нет AMSI), runspace API из .NET. Детект: EID 400 (engine start) с версией 2.

PowerShell: обнаружение обфускации в логах|Признаки в EID 4104: Invoke-Expression (iex), [System.Convert]::FromBase64String, [char] конкатенация, iex($env:комп), -bxor операции, gzip декомпрессия, много переменных для кусков кода. Инструмент: Revoke-Obfuscation — анализирует PS код на обфускацию. AMSI: Microsoft.PowerShell.Commands.UtilityCommon → AMSI scan. Правило: любой Base64 в PS командах = проверить декод.

PowerShell для анализа логов — быстрые команды|Get-WinEvent с XML фильтром: $filter = @{LogName='Security';Id=4625;StartTime=(Get-Date).AddHours(-1)}; Get-WinEvent -FilterHashtable $filter. Поиск brute force: Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625} | Group-Object {$_.Properties[19].Value} | Sort Count -Desc. Пример: Select-String "Failed password" /var/log/auth.log → grep аналог в PS: Select-String -Path "*.log" -Pattern "error".

---

## ══ BASH ДЛЯ SOC ══

Bash: анализ auth.log — топ команд|Топ IP с неудачными входами SSH: grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn | head -20. Успешные входы после неудач: grep "Accepted" /var/log/auth.log. Уникальные пользователи в sudo: grep "sudo:" /var/log/auth.log | awk '{print $6}' | sort -u. Активность за последний час: grep "$(date --date='1 hour ago' '+%b %d %H')" /var/log/auth.log.

Bash: анализ web-логов|Топ IP в nginx/apache: awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head. Топ URI: awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head. 404 ошибки: awk '$9==404' /var/log/nginx/access.log | awk '{print $7}' | sort | uniq -c | sort -rn. POST запросы: awk '$6=="\"POST"' /var/log/nginx/access.log | awk '{print $7}' | sort | uniq -c. Поиск сканера: grep -i "sqlmap\|nikto\|nessus\|masscan" /var/log/nginx/access.log.

Bash: поиск артефактов на хосте|Запущенные процессы с сетевыми соединениями: ss -tulnp | awk '{print $7}'. Файлы изменённые за последние 24ч в /tmp: find /tmp -mtime -1 -type f. SUID файлы: find / -perm -4000 -type f 2>/dev/null. Cron задачи: crontab -l; cat /etc/crontab; ls /etc/cron.d/. Подозрительные процессы: ps aux | grep -v "^root\|^USER" | awk '{if($3>50)print}'. Открытые файлы удалённого процесса: lsof -p PID.

Bash: сетевой анализ на хосте|Активные соединения с удалёнными хостами: ss -antp | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn. Прослушивающие порты: ss -tulnp | grep LISTEN. DNS резолюция IP: dig -x 1.2.3.4. Маршрут до IP: traceroute 1.2.3.4. Захват трафика: tcpdump -i any -nn -w /tmp/cap.pcap & sleep 30 && kill %1. Фильтр tcpdump: tcpdump -i eth0 'port 443 and host 1.2.3.4'.

Bash: быстрый IOC поиск по логам|Поиск IP из списка в логах: grep -F -f ioc_ips.txt /var/log/nginx/access.log. Поиск хешей: find / -type f -exec md5sum {} \; 2>/dev/null | grep -F -f known_bad_hashes.txt. Поиск доменов C2: cat /var/log/nginx/access.log | awk '{print $7}' | grep -F -f c2_domains.txt. Параллельный поиск: grep -rP "(malicious|pattern)" /var/log/ --include="*.log".

---

## ══ THREAT INTELLIGENCE ══

IOC типы — классификация|Atomic (атомарные): IP адрес, домен, хеш файла, URL, email адрес — не изменяются. Computed (вычисляемые): JA3 fingerprint, YARA правило, мьютекс имя, named pipe. Behavioral (поведенческие): TTP — наборы действий (MITRE техники) — самые ценные, так как не меняются при смене инструментов. Pyramid of Pain: IoCs снизу (легко менять), TTPs сверху (сложно менять).

TI Feeds — открытые источники|VirusTotal: проверка хешей, URL, IP, доменов (файлы → поведенческий анализ). AbuseIPDB: репутация IP адресов (брутфорс, спам). ThreatFox (abuse.ch): IOCs малвари (хеши, IP C2, URL). URLHaus (abuse.ch): вредоносные URL. MalwareBazaar: семплы малвари с хешами. MISP: open-source платформа TI (делиться IOCs). OTX (AlienVault): открытые "пульсы" с IOCs. Shodan: поиск устройств в интернете (разведка инфраструктуры атакующего).

TI Feeds — коммерческие|Recorded Future: предиктивная TI, ранние предупреждения. CrowdStrike Falcon Intelligence: ATG (adversary tracking), TTP-профили групп. Mandiant Advantage: профили APT групп, IOCs из расследований. Kaspersky TIP (Threat Intelligence Portal): хеши, C2, IoCs. IBM X-Force Exchange. BI.ZONE Threat Intelligence: собственный TI feed (знать для вакансии!). Интеграция: TAXII/STIX для обмена.

STIX/TAXII — что такое|STIX (Structured Threat Information Expression): стандартный формат описания TI (JSON). Объекты: Indicator (IOC + паттерн), Threat Actor (APT группа), Malware, Attack Pattern (TTP), Campaign, Course of Action. TAXII (Trusted Automated eXchange of Indicator Information): протокол обмена STIX данными между организациями. MISP использует STIX/TAXII для интеграций.

APT группы — знать ключевые|APT28 (Fancy Bear, Россия): правительство, военные, SOFACY. APT29 (Cozy Bear, Россия): долгосрочный шпионаж (SolarWinds). Lazarus Group (Северная Корея): финансовые кражи, шифрование, WannaCry. APT41 (Китай): шпионаж + финансовая мотивация. Conti (Россия-аффилированные): ransomware как сервис (RaaS). FIN7 (Carbanak): финансовые компании, PoS терминалы. BlackMatter/ALPHV: ransomware группы. Для SOC: маппинг инцидента → известная группа → её TTP.

TI жизненный цикл|1. Direction: определить что нужно знать (цели организации, топ угрозы). 2. Collection: сбор данных (OSINT, HUMINT, коммерческие фиды, dark web). 3. Processing: нормализация, дедупликация, обогащение. 4. Analysis: анализ, оценка достоверности (A1-F6 шкала Admiral). 5. Dissemination: доставка нужным командам (SIEM, SOC, C-suite). 6. Feedback: оценка полезности → корректировка. L1 участвует: 3,4 частично.

Proactive vs Reactive TI использование|Reactive: IOC пришёл → ищем в логах была ли компрометация. Проверить все системы за последние N дней на соответствие IOC. Proactive (Threat Hunting): гипотеза о TTP → активный поиск в данных без алерта. Пример: "APT28 использует WMI для lateral movement" → Hunt на WmiPrvSE.exe → child process в нашей среде. L1 в основном Reactive, L2/Hunting team — Proactive.

---

## ══ УПРАВЛЕНИЕ УЯЗВИМОСТЯМИ ══

CVE — структура и где смотреть|Common Vulnerabilities and Exposures. Формат: CVE-ГОД-НОМЕР (CVE-2021-44228 = Log4Shell). Описание включает: уязвимый продукт + версии, тип уязвимости, CWE (тип слабости). Базы: NVD (National Vulnerability Database): nvd.nist.gov — официальные CVE с CVSS. MITRE CVE: cve.mitre.org. Vendor Advisories: Microsoft Patch Tuesday, Cisco Security Advisories. ExploitDB: эксплойты к CVE.

CVSS v3.1 — как рассчитывается оценка|Common Vulnerability Scoring System. Метрики Base Score (0-10): AV (Attack Vector: N=Network, A=Adjacent, L=Local, P=Physical), AC (Attack Complexity: L/H), PR (Privileges Required: N/L/H), UI (User Interaction: N/R), S (Scope: U=Unchanged, C=Changed), C/I/A (Confidentiality/Integrity/Availability Impact: N/L/H). Critical: 9.0-10.0. High: 7.0-8.9. Medium: 4.0-6.9. Low: 0.1-3.9.

CVSS: как SOC использует оценку|Приоритизация патчинга: Critical + эксплойт в публичном доступе = немедленно. CVSS 10.0 сам по себе не означает критичность для вас (AV=Network но сервис не экспонирован = меньший реальный риск). EPSS (Exploit Prediction Scoring System): вероятность эксплуатации в 30 дней (0-100%). CISA KEV (Known Exploited Vulnerabilities): список CVE активно эксплуатируемых = ОБЯЗАТЕЛЬНЫЙ патч. Всегда смотреть: CVE + контекст (экспонирован ли сервис? есть ли публичный exploit?).

Patch Management цикл|1. Discovery: сканирование активов (Nessus, OpenVAS, MaxPatrol 8). 2. Prioritization: CVSS + EPSS + KEV + бизнес-критичность. 3. Testing: тестирование патча в dev/staging среде. 4. Deployment: rolling out (сначала некритичные, потом критичные). 5. Verification: повторное сканирование. 6. Reporting: метрики покрытия. Microsoft Patch Tuesday: каждый второй вторник месяца. SOC задача: алертить на уязвимые версии сервисов из сканирования vs новых CVE.

Nessus/OpenVAS — основы|Nessus (Tenable): коммерческий сканер уязвимостей. Плагины обновляются ежедневно. Типы сканирования: credentialed (с логином — полный аудит), network scan (без логина — сетевая доступность). OpenVAS (Greenbone): open-source альтернатива. MaxPatrol 8 (Positive Technologies): популярен в России, compliance + vulnerability scanning. SOC использует: результаты сканирования для контекста при расследовании (уязвим ли атакованный хост?).

Zero-Day vs N-Day — разница|Zero-Day: уязвимость для которой нет патча — производитель не знает или только узнал. Наиболее опасна: нет защиты кроме WAF/IPS правил и поведенческого детекта. Цена: $50K-$2.5M на рынке (браузерные, iOS). N-Day: известная уязвимость с доступным патчем (N дней назад). Большинство реальных атак — N-Day! Patch window: среднее время патчинга 30-60 дней = окно для атак. SOC: мониторить CISA KEV = реально эксплуатируемые.

---

## ══ ДОПОЛНИТЕЛЬНЫЕ СЕТИ — ПРОТОКОЛЫ ══

DNS записи типов — расширенно|A: IPv4 адрес. AAAA: IPv6. CNAME: псевдоним → другой домен (нельзя на root домен!). MX: mail сервер + приоритет. TXT: произвольный текст (SPF, DKIM, domain verification). NS: nameservers домена. SOA: Start of Authority (параметры зоны). PTR: обратная резолюция IP→домен. SRV: сервис + порт (Kerberos: _kerberos._tcp.domain). CAA: какой CA может выдавать сертификаты. DNSKEY/DS: DNSSEC.

DNS резолюция — полный путь|1. Браузер → OS кеш. 2. OS → /etc/hosts (или C:\Windows\System32\drivers\etc\hosts). 3. OS → Recursive Resolver (обычно ISP или 8.8.8.8). 4. Resolver → Root Servers (.). 5. Root → TLD (.com NS). 6. TLD → Authoritative NS домена. 7. Authoritative → ответ. Кеш на каждом уровне (TTL). SOC: hosts файл может перенаправить трафик (malware модифицирует), низкий TTL = DGA или C2 смена инфраструктуры.

BGP и атаки на маршрутизацию|BGP (Border Gateway Protocol): протокол маршрутизации между автономными системами (AS). AS = организация/ISP с блоком IP. BGP hijacking: AS анонсирует чужой IP префикс → трафик перенаправляется (инцидент с Ростелеком 2020, Pakistan Telecom YouTube 2008). Детект: BGPmon, RIPE NCC мониторинг, ROA (Route Origin Authorization) + RPKI защита. SOC: если исходящий трафик идёт неожиданным путём — проверить BGP.

Протокол DHCP детально — опции|DHCP выдаёт не только IP. Опции: 1=Subnet Mask, 3=Default Gateway, 6=DNS Servers, 15=Domain Name, 42=NTP Servers, 43=Vendor Specific, 51=Lease Time. DHCP Discover: broadcast UDP 67 (от клиента). DHCP Offer: unicast/broadcast от сервера. DHCP Request: клиент принимает offer. DHCP ACK: сервер подтверждает. Rogue DHCP: даёт свой gateway → все запросы через атакующего. Защита: DHCP Snooping.

NTP — зачем критичен для SOC|Network Time Protocol (UDP 123). Синхронизация времени на всех устройствах. Без NTP: временные метки событий расходятся → невозможна корреляция в SIEM. Kerberos: ±5 минут расхождения = аутентификация отказывает (защита от replay). NTP амплификация: DDoS атака (отправка MON_GETLIST запросов со spoofed IP). Источники: pool.ntp.org, DC как NTP источник в Windows домене (PDC Emulator → внешний NTP).

Разница: IDS vs IPS vs WAF vs Firewall|Firewall: фильтрация трафика по правилам (IP/порт/протокол). IDS (Intrusion Detection System): пассивный — обнаруживает + логирует, не блокирует. IPS (Intrusion Prevention System): активный — обнаруживает + блокирует. WAF (Web Application Firewall): специализирован на HTTP/HTTPS, понимает L7 (SQL Injection, XSS). NGFW = Firewall + IPS + App Control. Расположение: IDS/IPS в разрыв или на копии трафика (SPAN/TAP).

Suricata/Snort правила — синтаксис|Формат: action protocol src_ip src_port direction dst_ip dst_port (options). Пример: alert tcp any any -> $HOME_NET 445 (msg:"SMB lateral movement attempt"; sid:1000001; rev:1;). Действия: alert, drop (IPS), log. Options: content (паттерн), pcre (регулярки), classtype, threshold, flow, flags. Suricata: alert tls any any -> any any (tls.ja3; content:"известный_хеш_C2"; sid:2000001;). Правила скачивают: Emerging Threats, Snort VRT.

PROXY типы|HTTP Proxy (Forward Proxy): клиент явно настраивает (браузер → proxy настройки). Transparent Proxy: клиент не знает (трафик перехватывается шлюзом). SOCKS5 Proxy: работает с любым TCP/UDP протоколом (не только HTTP). Reverse Proxy: перед серверами. SSL Inspection Proxy (MITM Proxy): расшифровывает HTTPS для контент-инспекции (нужен корпоративный CA на клиентах). SOC: без SSL inspection = blind spot в 80%+ трафика сегодня. BYOD проблема: мобильные устройства без корп. CA.
