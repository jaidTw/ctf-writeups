## flagcrypt 100 (crypto)

### Solution

Send a message with length >= 20 to the server, and the server will reponse with the encrypted message and its length.
Through observing the provided code, the length of encrypted message will be shorter if there is repeated pattern (length > 3) occur in the plaintext, thus the encryption is vulnerable to [compression side channel attacks](https://www.sjoerdlangkemper.nl/2016/08/23/compression-side-channel-attacks/).

The charset is telled in the description, this exploit simply bruteforce the prodcuts of these characters with length = 3 first. Once a pattern found, it will try to extended it with one character at once.

A problem is there are 3 bytes patterns with common prefix or subfix ("me_", "ve_") and ("e\_d", "e\_a"), thus I manually change the flag variable and run the script again.

```python
#!/usr/bin/env python3
from pwn import *
import itertools


charset = set("abcdefghijklmnopqrstuvwxyz_")
r = remote("crypto.chal.csaw.io", 8041)
payload = "^" * 20

r.writelineafter("service\n", payload + "///")
data = r.recvline().strip()
data, sz = data[:-1], int(data[-1])

minsz = sz
flag = "me_doesnt_have_a"

def front_extend():
    global flag
    p = flag[:2]
    lst = []
    for c in charset:
        guess = payload + c + p
        r.writelineafter("service\n", guess)
        data = r.recvline().strip()
        data, sz = data[:-1], int(data[-1])
        lst.append((sz, c))
    minsz, c = min(lst, key=lambda t: t[0])
    maxsz, _ = max(lst, key=lambda t: t[0])
    lst = list(filter(lambda t: t[0] == minsz, lst))
    if minsz == maxsz:
        return
    if len(lst) > 1:
        print("Multiple possibilities :", lst)
    else:
        flag = c + flag
        print("flag : %s ... " % flag)
        front_extend()


def back_extend():
    global flag
    p = flag[-2:]
    lst = []
    for c in charset:
        guess = payload + p + c
        r.writelineafter("service\n", guess)
        data = r.recvline().strip()
        data, sz = data[:-1], int(data[-1])
        lst.append((sz, c))
    minsz, c = min(lst, key=lambda t: t[0])
    maxsz, _ = max(lst, key=lambda t: t[0])
    lst = list(filter(lambda t: t[0] == minsz, lst))
    if minsz == maxsz:
        return
    if len(lst) > 1:
        print("Multiple possibilities :", lst)
    else:
        flag += c
        print("flag : %s ... " % flag)
        back_extend()

for p in itertools.product(charset, repeat=3):
    p = "".join(p)
    guess = payload + p
    r.writelineafter("service\n", guess)
    data = r.recvline().strip()
    data, sz = data[:-1], int(data[-1])
    print(guess, sz)
    if sz < minsz:
        flag = p
        break

print("found %s, extending..." % flag)
front_extend()
back_extend()
```

### Flag

```
crime_doesnt_have_a_logo
```
