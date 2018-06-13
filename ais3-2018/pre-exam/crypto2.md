## XOR
### solution
先分析原本的script，得知key是一個8~12bytes的亂數，extend的行為基本上是將原本的key不斷重複擴展到長度L。

最後xor時，`flag+key`和`extend(key, len(flag+key))`會每個byte分別做XOR。而已知cipher和flag的前5 bytes(`AIS3{`)，因此可以先還原出key的前5bytes，接著從不同位址開始XOR觀察規律
```python
plain_prefix = b"AIS3{"
key_prefix = xor(cipher, plain_prefix)
print(key_prefix)
for i in range(0, len(cipher) - len(plain_prefix)):
    print(i, xor(key_prefix, cipher[i:]))
```
發現i為10的倍數時結果為純ASCII
```
0 b'AIS3{'
10 b'In aM'
20 b' - Wh'
30 b'R Hap'
40 b't0mOR'
50 b'OU mU'
60 b'0Mis3'
70 b'n3 tH'
80 b'TH4T '
90 b'iLL s'
100 b'ho Y0'
110 b'. Not'
120 b'Rfect'
130 b'IER, '
140 b' gOOD'
150 b'}\x16\t|\xc7'
```
因此我們可以得知key的長度為10，且第151個byte是`}`，將cipher長度減掉key長度也剛好為151，因此可以得知flag長度為151。
接著將xor對應關係大致繪圖如下
```
[AIS3{     FLAG             ][K  E  Y]
[K  E  Y][K  E  Y][K  E  Y][K  E  Y][K
[              CIPHER                ]
```
可以推得如下表，尾段的xor關係

| $p_{151}$| $k_{1}$ | $k_{2}$ | $k_{3}$ | $k_{4}$ | $k_{5}$ | $k_{6}$ | $k_{7}$ | $k_{8}$ | $k_{9}$ | $k_{10}$ |
|-|-|-|-|-|-|-|-|-|-|-|
| $k_{1}$| $k_{2}$ | $k_{3}$ | $k_{4}$ | $k_{5}$ | $k_{6}$ | $k_{7}$ | $k_{8}$ | $k_{9}$ | $k_{10}$ | $k_{1}$ |
| $c_{151}$ | $c_{152}$ | $c_{153}$ | $c_{154}$ | $c_{155}$ | $c_{156}$ | $c_{157}$ | $c_{158}$ | $c_{159}$ | $c_{160}$ | $c_{161}$ |

由於$k$的前5 bytes已知，我們就可以從$k_6$開始往後回推出完整的key
```
k[i] = k[i-1] XOR c[150+i] 
```

### script
```python
#!/usr/bin/env python3

with open('flag-encrypted', 'rb') as data:
    cipher = data.read()

def extend(key, L):
    kL = len(key)
    return key * (L // kL) + key[:L % kL]

def xor(X, Y):
    return bytes([x ^ y for x, y in zip(X, Y)])

plain_prefix = b"AIS3{"
key_prefix = xor(cipher, plain_prefix)

key = list(key_prefix) + [0, 0, 0, 0, 0]
for i in range(5, 10):
    key[i] = cipher[150 + i] ^ key[i - 1]

key = bytes(key)
print(key)
print(xor(cipher, extend(key, 151)))
```
### flags
```
AIS3{captAIn aMeric4 - Wh4T3V3R HapPenS t0mORr0w YOU mUst PR0Mis3 ME on3 tHIng. TH4T yOu WiLL stAY Who Y0U 4RE. Not A pERfect sO1dIER, buT 4 gOOD MAn.}
```
