## Special Instructions (Reverse)

### Solution

`file` the binary, notice that it's unknwon architecture.
```
$ file runme
runme: ELF 32-bit MSB executable, *unknown arch 0xdf* version 1 (SYSV), statically linked, not stripped
```

Use `readelf` instead
```
$ readelf -h runme
ELF Header:
  Magic:   7f 45 4c 46 01 02 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, big endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Moxie
  Version:                           0x1
  Entry point address:               0x1400
  Start of program headers:          52 (bytes into file)
  Start of section headers:          1936 (bytes into file)
  Flags:                             0x0
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         3
  Size of section headers:           40 (bytes)
  Number of section headers:         9
  Section header string table index: 8
```

It outputs its architecture is [Moxie](http://moxielogic.org/blog/).

I've tried `qemu-system-moxie` to run this file, but it hangs after started. Then I compiled `binutils` for target `moxie-unknown-elf`.

Now, use `objdump` and `nm` to get some infromation.
```
0000154a <set_random_seed>:
    154a:       16 20           bad
    154c:       04 00           ret

0000154e <get_random_value>:
    154e:       17 20           bad
    1550:       04 00           ret

00001552 <decode>:
    1552:       06 18           push    $sp, $r6
    1554:       06 19           push    $sp, $r7
    1556:       06 1a           push    $sp, $r8
    1558:       06 1b           push    $sp, $r9
    155a:       06 1c           push    $sp, $r10
    155c:       06 1d           push    $sp, $r11
    155e:       91 18           dec     $sp, 0x18
    1560:       02 d2           mov     $r11, $r0
    1562:       1c 42           ld.b    $r2, ($r0)
    1564:       2e 22           xor     $r0, $r0
    1566:       0e 42           cmp     $r2, $r0
    1568:       c0 12           beq     158e <decode+0x3c>
    156a:       02 a3           mov     $r8, $r1
    156c:       02 9d           mov     $r7, $r11
    156e:       01 c0 00 00     ldi.l   $r10, 0x154e
    1572:       15 4e
    1574:       1c 8a           ld.b    $r6, ($r8)
    1576:       2e 22           xor     $r0, $r0
    1578:       19 c0           jsr     $r10
    157a:       2e 82           xor     $r6, $r0
    157c:       1c 29           ld.b    $r0, ($r7)
    157e:       2e 82           xor     $r6, $r0
    1580:       1e 98           st.b    ($r7), $r6
    1582:       89 01           inc     $r7, 0x1
    1584:       8a 01           inc     $r8, 0x1
    1586:       1c 39           ld.b    $r1, ($r7)
    1588:       2e 22           xor     $r0, $r0
    158a:       0e 32           cmp     $r1, $r0
    158c:       c7 f3           bne     1574 <decode+0x22>
    158e:       02 2d           mov     $r0, $r11
    1590:       02 e0           mov     $r12, $fp
    1592:       9e 18           dec     $r12, 0x18
    1594:       07 ed           pop     $r12, $r11
    1596:       07 ec           pop     $r12, $r10
    1598:       07 eb           pop     $r12, $r9
    159a:       07 ea           pop     $r12, $r8
    159c:       07 e9           pop     $r12, $r7
    159e:       07 e8           pop     $r12, $r6
    15a0:       04 00           ret

000015a2 <main>:
    15a2:       06 18           push    $sp, $r6
    15a4:       91 18           dec     $sp, 0x18
    15a6:       01 20 92 d6     ldi.l   $r0, 0x92d68ca2
    15aa:       8c a2
    15ac:       03 00 00 00     jsra    154a <set_random_seed>
    15b0:       15 4a
    15b2:       01 80 00 00     ldi.l   $r6, 0x1480
    15b6:       14 80
    15b8:       01 20 00 00     ldi.l   $r0, 0x1
    15bc:       00 01
    15be:       01 30 00 00     ldi.l   $r1, 0x1654
    15c2:       16 54
    15c4:       19 80           jsr     $r6
    15c6:       01 20 00 00     ldi.l   $r0, 0x1
    15ca:       00 01
    15cc:       01 30 00 00     ldi.l   $r1, 0x1680
    15d0:       16 80
    15d2:       19 80           jsr     $r6
    15d4:       01 20 00 00     ldi.l   $r0, 0x1
    15d8:       00 01
    15da:       01 30 00 00     ldi.l   $r1, 0x169c
    15de:       16 9c
    15e0:       19 80           jsr     $r6
    15e2:       01 20 00 00     ldi.l   $r0, 0x1
    15e6:       00 01
    15e8:       01 30 00 00     ldi.l   $r1, 0x16ac
    15ec:       16 ac
    15ee:       19 80           jsr     $r6
    15f0:       01 20 00 00     ldi.l   $r0, 0x1
    15f4:       00 01
    15f6:       01 30 00 00     ldi.l   $r1, 0x16c4
    15fa:       16 c4
    15fc:       19 80           jsr     $r6
    15fe:       01 20 00 00     ldi.l   $r0, 0x1
    1602:       00 01
    1604:       01 30 00 00     ldi.l   $r1, 0x16e0
    1608:       16 e0
    160a:       19 80           jsr     $r6
    160c:       01 20 00 00     ldi.l   $r0, 0x1800
    1610:       18 00
    1612:       01 30 00 00     ldi.l   $r1, 0x1820
    1616:       18 20
    1618:       03 00 00 00     jsra    1552 <decode>
    161c:       15 52
    161e:       02 32           mov     $r1, $r0
    1620:       01 20 00 00     ldi.l   $r0, 0x1
    1624:       00 01
    1626:       19 80           jsr     $r6
    1628:       01 20 00 00     ldi.l   $r0, 0x1
    162c:       00 01
    162e:       01 30 00 00     ldi.l   $r1, 0x167c
    1632:       16 7c
    1634:       19 80           jsr     $r6
    1636:       2e 22           xor     $r0, $r0
    1638:       03 00 00 00     jsra    144a <exit>
    163c:       14 4a
```

There are two special functions, `set_random_seed` and `get_random_value`, which contains `bad` instructions.

I `strings` the file later and found some descriptions.
```
$ strings runme
,.U7
0123456789abcdef
This program uses special instructions.
SETRSEED: (Opcode:0x16)
        RegA -> SEED
GETRAND: (Opcode:0x17)
        xorshift32(SEED) -> SEED
        SEED -> RegA
GCC: (GNU) 4.9.4
moxie-elf.c
```

Now, with these information we can manually decompile the program.
```c
#define SYS_stdout 1

char *flag = (char *)0x1800;
char *ranval = (char *)0x1820;

int main() {
    set_random_seed(0x92d68ca2);
    puts(SYS_stdout, 0x1654);
    puts(SYS_stdout, 0x1680);
    puts(SYS_stdout, 0x169c);
    puts(SYS_stdout, 0x16ac);
    puts(SYS_stdout, 0x16c4);
    puts(SYS_stdout, 0x16e0);
    puts(SYS_stdout, decode(flag, randval));
    puts(SYS_stdout, 0x167c);
    exit(0);
}

char *decode(char *a, char *b) {
    char *pa = a;
    char *pb = b;
    if(a[0] == 0) return 0;
    for(;*pa; pa++, pb++) {
      *pa ^= *pb ^ get_random_value();
    }
    return a;
}
```

It's clear that `flag` will be xored with `randval` and a random value comes from `get_random_value()`, which is a xorshift32 PRNG.

So I write a script simulate the PRNG to decrypt the flag.

But... I've been spending more than one hours and can't get the flag, the generated sequence of my RNG isn't identical to the problem. Finally I notice that there's a version of pseudocode with different shifts on [Japanese Wikipedia](https://ja.wikipedia.org/wiki/Xorshift), I tried this version and successfully get the flag...


```python
#!/usr/bin/env python3

seed = 0x92d68ca2
mask = 0xFFFFFFFF

def get_rand():
    global seed
    seed ^= (seed << 13) & mask
    seed ^= (seed >> 17) & mask
    seed ^= (seed << 15) & mask
    return seed

with open('runme', 'rb') as f:
    f.seek(0x384)
    flag = f.read(0x20)
    f.seek(0x3a4)
    randval = f.read(0x20)

for i in range(32):
    r = get_rand() & 0xFF
    print(chr((flag[i] ^ randval[i] ^ r)), end="")
```

The flag is `SECCON{MakeSpecialInstructions}`
