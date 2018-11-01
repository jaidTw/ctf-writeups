## Mnemonic (Crypto)

### Solution

This challenge give us some text:
```
{
    "japanese": [
	[
	    "d3a02b9706507552f0e70709f1d4921275204365b4995feae1d949fb59c663cc",
	    "ふじみ　あさひ　みのう　いっち　いがく　とない　はづき　ますく　いせえび　たれんと　おとしもの　おどろかす　ことし　おくりがな　ちょうし　ちきゅう　さんきゃく　こんとん　せつだん　ちしき　ぬいくぎ　まんなか　たんい　そっと",
	    "338c161dbdb47c570d5d75d5936e6a32178adde370b6774d40d97a51835d7fec88f859e0a6660891fc7758d451d744d5d3b1a1ebd1123e41d62d5a1550156b1f"
	],
	[
	    "dfc9708ac4b4e7f67be6b8e33486482cb363e81967a1569c6fd888b088046f7c",
	    "ほんやく　ごうきゅう　おさめる　たこやき　ごかん　れいぎ　やせる　ふるい　まんなか　てんない　だんろ　さうな　きぼう　よくぼう　しのぐ　よけい　こんき　みうち　らくご　いわかん　いこく　あたためる　のはら　たぶん",
	    "bdadda5bbff97eb4fda0f11c7141bc3ce3de0fef0b2e4c47900858cec639c10187aee4695b1ba462b1dd34b170b62801e68c270b93af62629f4964947a620ed9"
	],
	[
	    "c0f...",
	    "???　とかす　なおす　よけい　ちいさい　さんらん　けむり　ていど　かがく　とかす　そあく　きあい　ぶどう　こうどう　ねみみ　にあう　ねんぐ　ひねる　おまいり　いちじ　ぎゅうにく　みりょく　ろしゅつ　あつめる",
	    "e9a..."
	],
    ],
    "flag": "SECCON{md5(c0f...)}"
}
```

After some googling, we found that it's [BIP 39 mnemonic code](https://iancoleman.io/bip39/).
Enter the second line as BIP 39 mnemonic, and we will get the third line, which is the seed.

But, what does the first line means? I read the [BIP 39 spec](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki), and found that an entropy is used to generate the mnenonic, thus I try to enter the first line as the entropy, and the output are identical to the other lines.

Finally, we have to recover the entropy of the last mnemonic codes, it's easy to bruteforce
```python
from mnemonic import *

l = [<list of japanese mnemonic codes>]

lg = Mnemonic("japanese")

for i in l:
    s = i+"　とかす　なおす　よけい　ちいさい　さんらん　けむり　ていど　かがく　とかす　そあく　きあい　ぶどう　こうどう　ねみみ　にあう　ねんぐ　ひねる　おまいり　いちじ　ぎゅうにく　みりょく　ろしゅつ　あつめる"
    if lg.to_seed(s).hex().startswith('e9a'):
        print(s)
        break
```

The first word of mnemonic is 「はいれつ」, now we have to tranform mnemonic codes back to entropy.
```python
l = [<list of japanese mnemonic codes>]
m = "はいれつ　とかす　なおす　よけい　ちいさい　さんらん　けむり　ていど　かがく　とかす　そあく　きあい　ぶどう　こうどう　ねみみ　にあう　ねんぐ　ひねる　おまいり　いちじ　ぎゅうにく　みりょく　ろしゅつ　あつめる".split("　")
list1=[]
for i in m:
        list1.append(l.index(i))
print(list1)
```
The output list is `[1543, 1333, 1376, 1953, 1173, 777, 570, 1262, 337, 1333, 995, 375, 1706, 616, 1485, 1404, 1495, 1644, 297, 91, 444, 1844, 2030, 24]`
Since each number is represeted by 11 bits, so we have to add some padding to trasform back to the original hexstring.
```python
s = [1543,1333,1376,1953,1173,777,570,1262,337,1333,995,375,1706,616,1485,1404,1495,1644,297,91,444,1844,2030,24]

b = "".join([bin(c)[2:].rjust(11, '0') for c in s])

b = "".join([hex(int(b[i:i+4], 2))[2:] for i in range(0, len(b), 4)])
print(b)
```

Finally, we get `c0f4d6b07a192ac251d4ee2a34d5f1977d549a2e6d7cbaf9b09485b379cd3f7018`, And the flag is `SECCON{cda2cb1742d1b6fc21d05c879c263eec}`
