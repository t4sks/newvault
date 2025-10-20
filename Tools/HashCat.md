Взлом хешей как john
base syntax 
```shell
hashcat -m <hash_mode> -a <attack_mode> <hashfile> <wordlist|mask> [опции]
```
`-m` - type of hash
`-a` - type of attack: 0 - wordlist; 1 - комбинатор; 3 - маска; 6/7 - гибрид
## Часто нужные режимы (-m)

- **16500** — JWT (HS256 секрет)
- **0** — MD5
- **100** — SHA1
- **1400** — SHA-256
- **1000** — NTLM
- **3200** — bcrypt ($2a$)
- **22000** — WPA/WPA2/WPA3 PMKID/EAPOL (hc22000)
- **17200** — ZIP (Legacy/ZipCrypto)
## Ключевые опции

- `--status --status-timer=10` — прогресс каждые 10 сек
- `--show` — показать найденные пары
- `--username` — игнорировать поле до `:` (если есть логины)
- `--force` — запуск при “капризном” драйвере (по необходимости)
- `--workload-profile 1..4` — нагрузка (по умолчанию 2/3)
- `--restore` / `--restore-timer=60` — продолжение прерванного сеанса
- `-r <rulefile>` — правила (мутации слов)
- `-S` — slow-candidates (умнее, медленнее)