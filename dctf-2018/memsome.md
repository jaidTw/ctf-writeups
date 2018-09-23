## Memsome (Reverse)

### Solution
1. There are many debugger detecting code, patch all of them first.
![](https://i.imgur.com/r43Xz3r.png)

2. Trace the program and found that it will keep reading a byte sequence and translate into a hex string with length 2240
![](https://i.imgur.com/bCQaGmD.png)

3. Each byte from the input will passthrough base64 and `sub_766C` twice, transform them into a 16bytes(32chars) string and compare with the string in step.2
![](https://i.imgur.com/66Vm3od.png)
![](https://i.imgur.com/ogcJeFg.png)

4. Collect the offset that data to be read from for string at step.2 first
```
0       0x7abb
32      0x7acc
64      0x7add
96      0xaee
128     0x7aff
160     0x7b10
192     0x7b21
224     0x7b21
256     hex1
288     0x7b59
320     0x7b10
352     0x7b6a
384     hex1
416     0x7b21
448     hex2
480     0x7b21
512     hex2
544     0x7ba1
576     0x7bb2
608     0x7bc3
640     0x7bb2
672     hex2
704     0x7ba1
736     0x7bc3
768     0x7b21
800     0x7bd4
832     0x7be5
864     hex2
896     0x7b59
928     0x7bd4
960     hex1
992     0x7bf6
1024    0x7c07
1056    0x7b21
1088    0x7be5
1120    0x7b59
1152    0x7c18
1184    0x7c07
1216    0x7bd4
1248    0x7bd4
1280    0x7bc3
1312    0x7bc3
1344    0x7bf6
1376    0x7bf6
1408    0x7c29
1440    0x7b6a
1472    0x7b21
1504    0x7bd4
1536    hex1
1568    0x7b6a
1600    0x7c18
1632    0x7bd4
1664    0x7be5
1696    0x7c29
1728    0x7bb2
1760    hex1
1792    0x7c29
1824    0x7c3a
1856    0x7b21
1888    hex2
1920    0x7bc3
1952    hex1
1984    0x7c3a
2016    0x7ba1
2048    0x7be5
2080    0x7bf6
2112    0x7c3a
2144    0x7bf6
2176    0x7c07
2208    0x7c4b
```

5. Use a script to generate the string in step.2
```
98678de32e5204a119a3196865cc7b83
e5a4dc5dd828d93482e61926ed59b4ef
68e8416fe8d00cca1950830c707f1e22
73747265616d4963545f4553355f6300
0b3dfc575614989f78f220e037543e55
75ac02c02f1f132e6c7314cad02f17cd
de32f4b8b17a37d87ea7436c6f215a34
de32f4b8b17a37d87ea7436c6f215a34
0061f1f351a4cddda4257550dc7d3000
0614aebdc5c356c2ca0f192c8f6880cb
75ac02c02f1f132e6c7314cad02f17cd
ddb8e3fc866990482e44ec0b78af08bd
0061f1f351a4cddda4257550dc7d3000
de32f4b8b17a37d87ea7436c6f215a34
cfff0050c0b39cf3bdde5a373b96b8a1
de32f4b8b17a37d87ea7436c6f215a34
cfff0050c0b39cf3bdde5a373b96b8a1
1e14bf5fdcbf5ec3945729ed48110d23
e4ad919d695a4ec3da99398d075aa21b
0f960e74a909d8558eefd9ab9ee8dbf3
e4ad919d695a4ec3da99398d075aa21b
cfff0050c0b39cf3bdde5a373b96b8a1
1e14bf5fdcbf5ec3945729ed48110d23
0f960e74a909d8558eefd9ab9ee8dbf3
de32f4b8b17a37d87ea7436c6f215a34
2933a2c8540feca890144972daad94c4
daa65a87e8a3d171b4d141c1ba716e49
cfff0050c0b39cf3bdde5a373b96b8a1
0614aebdc5c356c2ca0f192c8f6880cb
2933a2c8540feca890144972daad94c4
0061f1f351a4cddda4257550dc7d3000
c97a9650ef8edf91d8c20734ec20112e
40a7e180ede04d75b827585e9a1a547d
de32f4b8b17a37d87ea7436c6f215a34
daa65a87e8a3d171b4d141c1ba716e49
0614aebdc5c356c2ca0f192c8f6880cb
6bd1e27cd9cd034bf3d1d39d3e75b29e
40a7e180ede04d75b827585e9a1a547d
2933a2c8540feca890144972daad94c4
2933a2c8540feca890144972daad94c4
0f960e74a909d8558eefd9ab9ee8dbf3
0f960e74a909d8558eefd9ab9ee8dbf3
c97a9650ef8edf91d8c20734ec20112e
c97a9650ef8edf91d8c20734ec20112e
9ab45911b5716c3104e44a8947be4cf6
ddb8e3fc866990482e44ec0b78af08bd
de32f4b8b17a37d87ea7436c6f215a34
2933a2c8540feca890144972daad94c4
0061f1f351a4cddda4257550dc7d3000
ddb8e3fc866990482e44ec0b78af08bd
6bd1e27cd9cd034bf3d1d39d3e75b29e
2933a2c8540feca890144972daad94c4
daa65a87e8a3d171b4d141c1ba716e49
9ab45911b5716c3104e44a8947be4cf6
e4ad919d695a4ec3da99398d075aa21b
0061f1f351a4cddda4257550dc7d3000
9ab45911b5716c3104e44a8947be4cf6
fc496ce3c3e2e04604c375f7edc3cbc4
de32f4b8b17a37d87ea7436c6f215a34
cfff0050c0b39cf3bdde5a373b96b8a1
0f960e74a909d8558eefd9ab9ee8dbf3
0061f1f351a4cddda4257550dc7d3000
fc496ce3c3e2e04604c375f7edc3cbc4
1e14bf5fdcbf5ec3945729ed48110d23
daa65a87e8a3d171b4d141c1ba716e49
c97a9650ef8edf91d8c20734ec20112e
fc496ce3c3e2e04604c375f7edc3cbc4
c97a9650ef8edf91d8c20734ec20112e
40a7e180ede04d75b827585e9a1a547d
44e18699a27596eddd71b7920a04864b
```

5. `sub_766C` is too complicated, I spend more than three hours analyze it and give up. But I finally realize that I just need to enumearte all characters and intercept their encoding result(`*$rax`) after the second `sub_766C` call using gdb, and build a table
```
"9cbed6266f60ca7e18c6c18b08ace144": "A",
"b72f3ce3edc16fc82ac190c670945e83": "B",
"e5a4dc5dd828d93482e61926ed59b4ef": "C",
"98678de32e5204a119a3196865cc7b83": "D",
"05121f25ccf7058476a5f7d0f43a10a5": "E",
"226c14d44cd4e179b24b33a4103963c2": "F",
"020e4eceeae7931c809992184401e2c0": "G",
"89d22413a6825bcaf4ab2b4d8bd5935b": "H",
"4f81f5c6cb5325b7c1c1f5e0f3f9cd7d": "I",
"d09355804552f10586e18fd9b7c73d7d": "J",
"4005b42e317e77e796ca032c4331e3b8": "K",
"4681ad30737f72a311e5e3b9044438cd": "L",
"2cff56a46bf98b85bed3a933bc8e0d9c": "M",
"8e5341c1bd5979fae65e883a28b67b4e": "N",
"af6ec03a0a07c3d80665ec5130e2e593": "O",
"aefb764a059808b589d733aca123f5de": "P",
"16490be59a6da3cadbf0db02050ae1b5": "Q",
"90ba3f664d530000626914f80f8b5272": "R",
"d83e98514ae6735092cad56fb8dc7b48": "S",
"68e8416fe8d00cca1950830c707f1e22": "T",
"871b45a56ea9687d28492550b41ad558": "U",
"543884539cecbe3bd4e78860dbf70cfe": "V",
"305c19426fecca73a7d72f4b31e0e92d": "W",
"c3202ec1672642fde3077971bbb45e73": "X",
"c88c8d7854b735bad26190a4636b0288": "Y",
"f01d3924eaa08676a8cb6bdab91ff06d": "Z",
"de32f4b8b17a37d87ea7436c6f215a34": "a",
"9ab45911b5716c3104e44a8947be4cf6": "b",
"40a7e180ede04d75b827585e9a1a547d": "c",
"ddb8e3fc866990482e44ec0b78af08bd": "d",
"6bd1e27cd9cd034bf3d1d39d3e75b29e": "e",
"0f960e74a909d8558eefd9ab9ee8dbf3": "f",
"979ac9caba8271dc1c6e4f6ee52ec2d0": "g",
"6d02afeb4553323f3e2537afc4900427": "h",
"4a39ba8a55375ead7773ba9d800bab01": "i",
"c7cd26b4f506085c2e954d7e290f179d": "j",
"3b35be578493524a2249888e71a8a3a7": "k",
"e65573b6babfcdc16045c91e88aed749": "l",
"e8c1e678e4902e12b83f8f75b1409933": "m",
"0b3dfc575614989f78f220e037543e55": "n",
"c1c2eae083eab5a8558b77a834988c4f": "o",
"8aceb753526e4ba898c5f343fd868e9e": "p",
"788bd3e083dda7e1459f9bd1b79990bf": "q",
"aea7c77980b7a22764702f2e173384b9": "r",
"7f0b16b72a9a64b957d6d8f3945508b6": "s",
"df6d46fb7df61c19d995430432970d55": "t",
"e67ec66f53b6924402cdc907d27deeda": "u",
"8af7278d5700c86e901628704741f0eb": "v",
"ed3554a4707539dc61c10661edda8aea": "w",
"d3135c717071ba3bf410d74caa62bb6d": "x",
"5a1cdc7e29df2bc486df10990cf81579": "y",
"5a1cdc7e29df2bc486df10990cf81579": "z",
"0061f1f351a4cddda4257550dc7d3000": "1",
"1e14bf5fdcbf5ec3945729ed48110d23": "2",
"c97a9650ef8edf91d8c20734ec20112e": "3",
"0614aebdc5c356c2ca0f192c8f6880cb": "4",
"e4ad919d695a4ec3da99398d075aa21b": "5",
"daa65a87e8a3d171b4d141c1ba716e49": "6",
"2933a2c8540feca890144972daad94c4": "7",
"cfff0050c0b39cf3bdde5a373b96b8a1": "8",
"75ac02c02f1f132e6c7314cad02f17cd": "9",
"fc496ce3c3e2e04604c375f7edc3cbc4": "0",
"2460ca5292277ff9076ab2c34a12f583": "_",
"0b3dfc575614989f78f220e037543e55": "{",
"44e18699a27596eddd71b7920a04864b": "}",
```

6. finally, use the table to substitue the string in step.5
```
DCT^{9aa149d1a8a825f582fa7684713ca64ec77ff33bda71de76b51b0a8f1026303c}
Lossing key:
{'73747265616d4963545f4553355f6300'}
```

7. One character is lossing, I guess it's `F`, subsitue it and it's correct!

### Exploit
```python
#!/usr/bin/env python3
import binascii

hex1 = "0061f1f351a4cddda4257550dc7d3000"
hex2 = "cfff0050c0b39cf3bdde5a373b96b8a1"

license = ""

encode_table = {
    # omitted here
}

with open('memsom', 'rb') as b:
    with open('offsets', 'r') as f:
        for line in f.readlines():
            _, offset = line.split()
            if offset == "hex1" or offset == "hex2":
                license += eval(offset)
            else:
                offset = int(offset[2:], 16)
                b.seek(offset)
                data = b.read(16)
                license += binascii.b2a_hex(data).decode()

ss = set()
print(len(license))
for i in range(0, len(license), 32):
    print(license[i:i+32])

for i in range(0, len(license), 32):
    try:
        print(encode_table[license[i:i+32]], end="")
    except KeyError:
        print("^", end="")
        ss.add(license[i:i+32])
print("\nLossing key:")
print(ss)

```

### Flag
```
DCTF{9aa149d1a8a825f582fa7684713ca64ec77ff33bda71de76b51b0a8f1026303c}
```
