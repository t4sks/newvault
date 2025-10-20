сканер для веба
fast scan 
```shell
feroxbuster -u http://<IP> -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt,bak,zip,tar,conf,inc -t 200 -s 200,204,301,302,403

```