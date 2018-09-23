## Ransomware (Reverse) 

### Solution

Unzip the file and get `youfool!.exe` and `ransomware.pyc`.
The exe file doesn't seems to be a real executable, use uncompyle6 decompile `ransomware.pyc` first:
```python
# uncompyle6 version 3.2.3
# Python bytecode 2.7 (62211)
# Decompiled from: Python 3.7.0 (default, Sep 15 2018, 19:13:07) 
# [GCC 8.2.1 20180831]
# Embedded file name: ransomware.py
# Compiled at: 2018-09-04 21:35:11
import string
from random import *
import itertools

def caesar_cipher(inp, code):
    code = code * (len(inp) / len(code) + 1)
    return ('').join((chr(ord(i) ^ ord(j)) for i, j in itertools.izip(inp, code)))


f = open('./FlagDCTF.pdf', 'r')
buf = f.read()
f.close()
allchar = string.ascii_letters + string.punctuation + string.digits
password = ('').join((choice(allchar) _ in range(60)))
buf = caesar_cipher(buf, password)
f = open('./youfool!.exe', 'w')
buf = f.write(buf)
f.close()
```
The code is already cleanup a bit, apparently, a pdf is XOR with the random string to generate the exe file, and we have to recover pdf.

The `password` has 60 bytes and it's difficult to bruteforce, but we can infer some bytes first based on the PDF file structure. There are many documents on the web, just search for them and started from the header and the trailer part, then try to find some text frgaments, keep extend them and we're done.

The final recover script:
```python
#!/usr/bin/env python3

def decode(cipher, code):
    code = code * (len(cipher) // len(code) + 1)
    return b"".join((bytes((i ^ ord(j),)) for i, j in zip(cipher, code)))

def dump(buf, offset, len):
    print(offset, end=" ")
    for i in range(len):
        print(hex(ord(buf[offset+i])), end=", ")
    print()

with open("youfool!.exe", "rb") as f:
    buf = f.read()

passwd = ['a' for _ in range(60)]
passwd[0] = chr(buf[0] ^ ord('%'))
passwd[1] = chr(buf[1] ^ ord('P'))
passwd[2] = chr(buf[2] ^ ord('D'))
passwd[3] = chr(buf[3] ^ ord('F'))
passwd[4] = chr(buf[4] ^ ord('-'))
passwd[5] = chr(buf[5] ^ ord('1'))
passwd[6] = chr(buf[6] ^ ord('.'))
passwd[7] = chr(buf[10507] ^ ord('e')) # /Type
passwd[8] = chr(buf[1388] ^ ord('e')) # /Filter
passwd[9] = chr(buf[1389] ^ ord('r'))
passwd[10] = chr(buf[8470] ^ ord('t')) # /Length
passwd[11] = chr(buf[8471] ^ ord('h'))
passwd[12] = chr(buf[8832] ^ ord('e')) # /Filter
passwd[13] = chr(buf[8833] ^ ord('r'))
passwd[14] = chr(buf[254] ^ ord('t')) # /Filter
passwd[15] = chr(buf[255] ^ ord('e'))
passwd[16] = chr(buf[256] ^ ord('r'))
passwd[17] = chr(buf[1397] ^ ord('D')) # /FlateDecode
passwd[18] = chr(buf[1398] ^ ord('e'))
passwd[19] = chr(buf[1399] ^ ord('c'))
passwd[20] = chr(buf[1400] ^ ord('o'))
passwd[21] = chr(buf[1401] ^ ord('d'))
passwd[22] = chr(buf[1402] ^ ord('e'))
passwd[23] = chr(buf[983] ^ ord('a')) # /Image
passwd[24] = chr(buf[984] ^ ord('g'))
passwd[25] = chr(buf[985] ^ ord('e'))

passwd[26] = chr(buf[1406] ^ ord('e')) # /Length
passwd[27] = chr(buf[1407] ^ ord('n'))

passwd[28] = chr(buf[988] ^ ord('/')) # /Image
passwd[29] = chr(buf[989] ^ ord('I'))
passwd[30] = chr(buf[990] ^ ord('m'))

passwd[31] = chr(buf[1051] ^ ord('/')) # /Flate
passwd[32] = chr(buf[1052] ^ ord('F'))
passwd[33] = chr(buf[10053] ^ ord('/')) # /Resources
passwd[34] = chr(buf[10054] ^ ord('R'))
passwd[35] = chr(buf[10055] ^ ord('e'))
passwd[36] = chr(buf[9996] ^ ord('/')) # /Contents
passwd[37] = chr(buf[9997] ^ ord('C'))
passwd[38] = chr(buf[9998] ^ ord('o'))
passwd[39] = chr(buf[1419] ^ ord('/')) # /Length
passwd[40] = chr(buf[1420] ^ ord('L'))
pad = len(buf) % 60 # pad = 47
passwd[pad-6] = chr(buf[-6] ^ ord('%'))
passwd[pad-5] = chr(buf[-5] ^ ord('%'))
passwd[pad-4] = chr(buf[-4] ^ ord('E'))
passwd[pad-3] = chr(buf[-3] ^ ord('O'))
passwd[pad-2] = chr(buf[-2] ^ ord('F'))
passwd[pad-1] = chr(buf[-1] ^ 0x0a)
passwd[47] = chr(buf[1067] ^ ord('n')) # /Length
passwd[48] = chr(buf[1068] ^ ord('g'))
passwd[49] = chr(buf[1069] ^ ord('t'))
passwd[50] = chr(buf[1070] ^ ord('h'))
passwd[51] = chr(buf[10551] ^ ord('o')) # /DecodeParms
passwd[-8] = chr(buf[892] ^ ord('/')) # /Resources
passwd[-7] = chr(buf[893] ^ ord('R'))
passwd[-6] = chr(buf[9534] ^ ord('/')) # /Resource
passwd[-5] = chr(buf[9535] ^ ord('R'))
passwd[-4] = chr(buf[9536] ^ ord('e'))
passwd[-3] = chr(buf[9537] ^ ord('s'))
passwd[-2] = chr(buf[9538] ^ ord('o'))
passwd[-1] = chr(buf[9539] ^ ord('u'))

passwd = "".join(passwd)
plain = decode(buf, passwd)
with open("out.pdf", "wb+") as f:
    f.write(plain)

for i in range(len(plain)//60 + 1):
    print(i*60, plain[i*60:i*60+60])
```

But the recoverd pdf seems still destroyed, it lacks of xref section, and can't opened by readers, I try to use `qpdf` to recover the file and finally success.
```
$ qpdf --qdf --object-streams=disable out.pdf new.pdf
```

Open the new pdf and you'll see the flag.

### Flag
```
DCTF{d915b5e076215c3efb92e5844ac20d0620d19b15d427e207fae6a3b894f91333}
```
