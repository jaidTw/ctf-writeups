## Shredder (misc)

### Solution

`floppy` is a FAT12 image, we can extract an ELF executable `shredder` and a deleted file `flag.txt` from it using the tool `fatcat`

Try to analyze the behaviour of shredder, I found
* usage `./shredder passes files`...
* Behaviour：Byte by byte XOR the contents of files with a random number generated by `getrandom()` with `passes` times, and unlink(delete) the file.

Have a look on the deleted `flag.txt`, it's encrypted, thus we can infer that `flag.txt` is deleted by `shredder`. What we have to do is simply try to XOR `flag.txt` with all possible values of a byte (0 ~ 255).

### Exploit

```python
#!/usr/bin/env python3

f = open("flag.txt")
s = f.read()
for i in range(255):
    print("".join([chr(ord(c) ^ i) for c in s]))
```

### Flag

```
SECT{1f_U_574y_r1gh7_wh3r3_U_R,_7h3n_p30pl3_w1ll_3v3n7u4lly_c0m3_70_U}
```
