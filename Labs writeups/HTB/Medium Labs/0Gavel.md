## Краткое описание

## Оглавление
[[#Разведка]]
[[#Анализ исходного кода]]
[[#SQL инъекция]]
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
по факту у нас 