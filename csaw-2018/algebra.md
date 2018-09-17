## algebra 50 (misc)

### Solution

We have to solve several equations in this challenge, all equations are one-dimensional about `X`, I simply transform the equation from the form `f = b` to `f - b` and solve it with SymPy.

```python
#!/usr/bin/env python3
from pwn import *
from sympy import Symbol, solve
r = remote("misc.chal.csaw.io", 9002)

r.recvuntil("*\n")
while True:
    eq = r.recvline().strip().decode()
    if eq.startswith("flag"):
        print(eq)
        break
    X = Symbol("X")
    eq = "solve(" + eq.replace("=", "-") + ")"
    print(eq)
    try:
        sol = float(eval(eq)[0])
        print("X =", sol)
    except IndexError:
        r.sendline("0")
    else:
        r.sendline(str(sol))
    r.recvuntil("going\n")
```

### Flag

```
flag{y0u_s0_60od_aT_tH3_qU1cK_M4tH5}
```
