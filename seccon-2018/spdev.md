## Special Device File (Reverse)

### Solution

This challenge is similar to [Special Instruction](./spins.md), but the binary is AArch64 ELF, thus we can use decompiler this time.

```c
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  unsigned int fd; // w21 MAPDST
  char *flag_buf; // x0
  __int64 seed; // [xsp+28h] [xbp-8h]

  seed = 0x139408DCBBF7A44LL;
  fd = _open("/dev/xorshift64", 1LL, 0LL);
  _write();
  _close(fd);
  fd = _open("/dev/xorshift64", 0LL, 0LL);
  flag_buf = decode(&flag, &randval, fd);
  puts(1LL, flag_buf);
  puts(1LL, "\n");
  _close(fd);
  exit(0LL);
}

char *__fastcall decode(char *flag, char *key, unsigned int rng)
{
  char *p; // x21
  __int64 i; // x3

  if ( *flag )
  {
    p = flag;
    i = 0LL;
    do
    {
      *p ^= get_random_value(rng) ^ key[i];
      ++i;
      ++p
    }
    while ( flag[i] );
  }
  return flag;
}
```

It's pretty simple. In `main` it will open `/dev/xorshift64`, write something then close it, IDA didn't correctly infer the argument here, we can assume it writes `0x139408DCBBF7A44LL` to the device, which is the seed of the RNG.

And then, in the `decode` function, a global encrypted `flag`(`0x1800`) will perform XOR with `randval`(`0x1820`) and `get_random_value`. Simply extract these bytes and simulate the RNG, then we can decrypt the flag.

Again, I spent many time on finding the correct version of xorshift64, and finally found it on [Japanese Wikipedia](https://ja.wikipedia.org/wiki/Xorshift).

```python
#!/usr/bin/env python3

seed = 0x139408DCBBF7A44
mask = 0xFFFFFFFFFFFFFFFF

def get_rand():
    global seed
    seed ^= (seed << 13) & mask
    seed ^= (seed >> 7) & mask
    seed ^= (seed << 17) & mask
    return seed

with open('runme', 'rb') as f:
    f.seek(0x1800)
    flag = f.read(0x20)
    f.seek(0x1820)
    randval = f.read(0x20)

for i in range(32):
    r = get_rand()
    print(chr((flag[i] ^ randval[i] ^ r) & 0xFF), end="")
```

The flag is `SECCON{UseTheSpecialDeviceFile}`
