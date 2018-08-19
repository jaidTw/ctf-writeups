## lenna
### solution
題目給了一個網頁，裡面有兩張圖片，右邊的那張分成數個td cell，某些點擊後會變成黑色，可以點擊的部份可以拼湊成一個字母R，以上沒有任何意義。

將左右兩張圖片下載下來進行比對
```sh
$ cmp -l Lenna_left.bmp Lenna_right.bmp | gawk '{printf "%08X %02X %02X\n", $1, strtonum(0$2), strtonum(0$3)}' > out
```
接著從結果可以發現類似flag的字母
```
00029437   g
0002949D A f
000294DF Y 5
0002954B n }
0002B837 y u
0002B89D E 8
0002B8DF S 8
0002B94B ` d
0002B987   8
0002B9C0 _ 5
0002BA29 _ d
0002BA65 m 5
0002BAB6 q d
0002BAD7 W f
0002BB37 q e
0002BB9D E f
0002BBDF V 8
0002BC4B X 8
0002BCC0 b w
0002BD29 ^ s
0002BD65 q 5
0002BDB6 w d
0002BDD7 R s
0002C737 x A
0002C79D J E
0002C7DF \ G
0002C84B \ I
0002C887 Y S
0002C8C0 c {
0002C929 g 0
0002C965 o 2
0002C9B6   3
0002C9D7 U d
```
然而順序有點怪異，需要重新排列，根據offset末兩位的循環來進行分割得到
```
gf5} u88d85d5df ef88ws5ds AEGIS{023d
```
反序排列得到
```
AEGIS{023def88ws5dsu88d85d5dfgf5}
```
即是正確flag
