фуззинг веба работает заменяя слово `FUZZ` в URL 
общий синтаксис 
```shell
ffuf -u "http://site.com/page.php?FUZZ=value" -w /path/to/wordlist.txt
```

---
use this example for check key words in php 
```shell
ffuf -u 'http://10.201.82.109/assets/index.php?FUZZ=id' -ac -mc all -t 200 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -fs 0
```
- `-mc all` — выводить все ответы.
- `-fs 0` — исключить ответы с нулевым телом (size == 0).
- `-ac` — авто-калибровка, отбрасывает повторяющиеся «шумные» ответы.
- `-t 50` — обезопасить целевой сервер (200 потоков не нужен в лабе).
- `-o` — сохраняет результат.
---
Брутфорс поддоменов
```shell
$ ffuf -u 'http://ip/' -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.ip" -mc all -t 200 
```