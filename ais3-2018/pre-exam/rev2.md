## secret
### solution
丟進IDA分析，程式會`srand(0)`接著連續85次和`rand() % 2018`比較來根據亂數把flag寫到`/tmp/secret`
題目也說明了gcc的版本(影響的應該是glibc?)，因此一樣設定seed產生相同的亂數餵給程式，最後看`/tmp/secret`即可。
### script
```c
/* crng.c */
#include <stdio.h>
#include <stdlib.h>
int main(void) {
    srand(0);
    for(;;) {
        printf("%d\n", rand());
        getchar();
    }
    return 0;
}
```
```python
#!/usr/bin/env python3
from pwn import *
class crng:
    def __init__(self):
        self.p = process("crng")
    def rand(self):
        i = int(self.p.recvline().strip())
        self.p.sendline()
        return i
r = process("secret")
rng = crng()
for _ in range(85):
    r.sendline(str(rng.rand() % 2018))
r.interactive()
```
### flag
```
AIS3{tHere_1s_a_VErY_VerY_VeRY_1OoO00O0oO0OOoO0Oo000OOoO00o00oG_f1@g_iN_my_m1Nd}
```
