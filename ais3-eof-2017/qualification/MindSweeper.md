## MindSweeper (400)
### solution
透過OllyDbg追蹤，試著去踩地雷，從`WinMain`開始一路step in會發現在`call 0x402660`前後畫面發生變化，在`0x402678`處原本為`je short 00402698`，將其試著patch為`jmp`後就不會踩到地雷，接著觀察地雷的pattern測試幾次之後發現是一個hexstring，一旦輸了就會重新開始，擷取前幾個
```
504B0304 14000000 08009188 224CE9D4 2022D6D5
0000007E 01000800 0000666C 61672E65 7865ECFD
0D5C9455 DA388EDF F3020C30 38A382A2 52626252
688B8E96 345A8C3A 88253648 DE92015A 423...
```
試著轉成text
```
PK����"LéÔ "ÖÕ���~����flag.exeìý
\UÚ8ßó08£¢RbbRh4Z:%6HÞZB4
```
會發現PK和`flag.exe`的字眼，而`504b0304`是PK zip的header，因此推測要把這段binary存出來再解壓縮

接著繼續IDA逆向，這部分花了很多時間，這邊只寫重點：
在string裡面會找到
```
HackerText db 'Are u a hacker O_o?',0
```
 這樣的字串，利用cross reference定位到`0x402860`的block反編譯，大概可以得到這樣的結果
```c
int __cdecl set_pattern()
{
  ...
  if ( win_count == 109784 )
  {
    MessageBoxA(0, HackerText, O_o, 0x20u);
    result = gen_sth(1);
    win_count = 0;
  }
  else
  {
    ...
    rand_x = get_rand(width - 4) + 1;
    rand_y = get_rand(length - 5) + 1;
    for ( k = 0; k < 16; ++k )
    {
      val = data1[32 * win_count + k] ^ data2[32 * win_count + k];
      if ( val == 255 )
        break;
      minearray[100 * ((val & 0xF) + rand_y) + (val >> 4) + rand_x] &= 0x7Fu;
    }
    ...
  }
  return result;
}
```
其中`win_count`是從其他routine中分析得知，只要每贏一次就會遞增1，因此推測hexstring的總長度為109784，接著可以看到下面的`data1`和`data2`運算後得到`val`，接著切割成x、y offset疑似用來設定地雷的位置，跳到資料的部分發現`data1`和`data2`是overlap的，而`data1 = data2 + 16bytes`。

試著用python script從binary的該位置(`0x146c8`)讀raw出來，然後一樣的方式進行xor運算，切割x、y，在8x8的list中把pattern印出來，發現和遊戲所得到的一致：
```python
#!/usr/bin/python
data = []

with open('winmine.exe', 'rb') as f:
    print(f)
    f.seek(0x146c8) # start offset of data
    data = f.read(0x76f3c7 - 0x4158c8) # range of data
    
for w in range(5): # test 5 words
    graph = [[0 for _ in range(8)] for _ in range(8)]
    for k in range(16):
        val = data[32 * w + k] ^ data[32 * w + k + 16]
        if val == 255:
            break
        graph[val & 0xf][val >> 4] = 1

    for i in range(8): # plot the pattern
        for j in range(8):
            if graph[i][j] == 1:
                print("X", end="")
            else:
                print(" ", end="")
        print()
```

因此，我們利用這些binary data就可以轉換回原本的zip，這裡作法是每個iteration(每個hex char)所得到的所有val相乘作為hash value，先跑一些資料把對應的hash table建出來，接著再利用這個table將結果轉為hex string的檔案，最後再將該檔案轉換成binary
```python
#!/usr/bin/python
import sys

hash_to_hex = {
        18012773987208000 : b'0',
        4948564282200 : b'1',
        115177833668205000 : b'2',
        499103945895555000 : b'3',
        22195177804800 : b'4',
        59892473507466600 : b'5',
        239569894029866400 : b'6',
        193187194200 : b'7',
        11978494701493320000 : b'8',
        2994623675373330000 : b'9',
        15416338097160000 : b'A',
        4793315206680000 : b'B',
        135842941080 : b'C',
        367607632392000 : b'D',
        90335555818200 : b'E',
        111874732200 : b'F',
        }

data = []
with open('winmine.exe', 'rb') as f:
    f.seek(0x146c8)
    data = f.read(0x76f3c7 - 0x4158c8)

buf = b""
try :
    with open('hexstr', 'wb') as f: # write result to hexstr
        print(len(data))
        for w in range(109784):
            h = 1
            for k in range(16):
                val = data[32 * w + k] ^ data[32 * w + k + 16]
                if val == 255:
                    break
                h *= (val + 1)
            buf += hash_to_hex[h]
            if len(buf) == 0x20:
                f.write(buf)
                f.write(b'\n')
                buf = b""
except IndexError:
    sys.stderr.buffer.write(b"index  error")
    pass

with open('hexstr', 'r') as f: # output bytes result to stdout
    data = f.readlines()
    for line in data:
        sys.stdout.buffer.write(bytes.fromhex(line.strip()))
```

之後就可以得到檔案，而試著解壓卻發現檔案不完整，但若是使用7zip則仍能將`flag.exe`解壓出來，接著執行`flag.exe`就可以取得flag了
