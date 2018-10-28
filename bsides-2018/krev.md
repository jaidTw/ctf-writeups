## krev (reverse, 200)

### Solution

What making this challenge complex is that it's wrapped in a NetBSD kernel module.

We're given a NetBSD VM with gdb to debug, but I simply copy the `chall1.kmod` from the VM.

Try to decompile the module, we'll se some functions.

* `chall1_close`
* `chall1_open`
* `chall1_write`
* `chall1_modcmd`
* `md5hash`
* `sha1hash`
* `get_flag_ready`
* `chall1_read`

We're intereseted in `chall1_read`

```c
int __cdecl chall1_read(int a1, int a2, int a3)
{
  int result; // eax
  char *v4; // ebx
  size_t v5; // eax
  size_t v6; // eax
  char s; // [esp+10h] [ebp-58h]

  if ( *buf != 'g'
    || buf[1] != 'i'
    || buf[2] != 'v'
    || buf[3] != 'e'
    || buf[4] != '_'
    || buf[5] != 't'
    || buf[6] != 'h'
    || buf[7] != 'i'
    || buf[8] != 's'
    || buf[9] != '_'
    || buf[10] != 't'
    || buf[11] != 'o'
    || buf[12] != '_'
    || buf[13] != 'g'
    || buf[14] != 'e'
    || buf[15] != 't'
    || buf[16] != '_'
    || buf[17] != 'f'
    || buf[18] != 'l'
    || buf[19] != 'a'
    || buf[20] != 'g' )
  {
    snprintf(&s, 0x19u, "%s", "Why don't you try again?");
    result = uiomove(&s, 24, a3);
  }
  else
  {
    get_flag_ready();
    v4 = flag;
    v5 = strlen(flag);
    snprintf(&s, v5 + 1, "%s", v4);
    v6 = strlen(flag);
    result = uiomove(&s, v6 + 1, a3);
  }
  return result;
}
```

Looks like, if `buf` equals `give_this_to_get_flag`, then it will call `get_flag_ready` and return the flag to user space.

Let's see what's in `get_flag_ready` next.

```c
size_t get_flag_ready()
{
  size_t i; // ebx
  size_t key_len; // eax
  char code[41]; // [esp+4h] [ebp-2Ch]

  code[0] = 0x56;
  code[1] = 0x5C;
  code[2] = 0x50;
  code[3] = 5;
  code[4] = 0x4D;
  code[5] = 0xF;
  code[6] = 0x53;
  code[7] = 0x47;
  code[8] = 0x76;
  code[9] = 0x57;
  code[10] = 0x21;
  code[11] = 0x3A;
  code[12] = 0x5E;
  code[13] = 6;
  code[14] = 0x3B;
  code[15] = 0xD;
  code[16] = 0x11;
  code[17] = 0x16;
  code[18] = 2;
  code[19] = 9;
  code[20] = 0xB;
  code[21] = 0x67;
  code[22] = 0x1B;
  code[23] = 0x52;
  code[24] = 0x41;
  code[25] = 0x6B;
  code[26] = 0x40;
  code[27] = 0x5D;
  code[28] = 0x56;
  code[29] = 0x17;
  code[30] = 0x5D;
  code[31] = 1;
  code[32] = 0x3A;
  code[33] = 4;
  code[34] = 0x13;
  code[35] = 0x1D;
  code[36] = 0x68;
  code[37] = 0x50;
  code[38] = 6;
  code[39] = 0x45;
  md5hash();
  sha1hash();
  for ( i = 0; ; ++i )
  {
    key_len = strlen(key);
    if ( i >= key_len )
      break;
    flag[i] = code[i] ^ key[i];
  }
  return key_len;
}
```

`flag` is computed by XOR `code` with `key`, but what is `key`?

```c
int md5hash()
{
  const char *data; // esi
  unsigned int len; // eax
  char *p; // esi
  char *output; // ebx
  int result; // eax
  char digest[16]; // [esp+10h] [ebp-70h]
  char ctx; // [esp+20h] [ebp-60h]

  MD5Init(&ctx);
  data = buf;
  len = strlen(buf);
  MD5Update(&ctx, data, len);
  p = digest;
  MD5Final(digest, &ctx);
  output = buf2;
  do
  {
    result = snprintf(output, 5u, "%02x", (unsigned __int8)*p++);
    output += 2;
  }
  while ( output != (char *)&unk_80006D4 );
  return result;
}
```

In `md5hash`, `buf` is hashed by md5 and written to `buf2`. In `sha1hash`, it will perform similar work, hashes `buf2` into `key`. Finally, now we know `key` is `sha1(md5(buf))`, so `flag = code ^ sha1(md5(buf))`.

Here's the script to decrypt it.

```python
#!/usr/bin/env python3
from hashlib import md5, sha1

key = b"give_this_to_get_flag"
key = md5(key).hexdigest().encode()
key = sha1(key).hexdigest().encode()

code =  b'\x56\x5c\x50\x05\x4d\x0f\x53\x47'
code += b'\x76\x57\x21\x3a\x5e\x06\x3b\x0d'
code += b'\x11\x16\x02\x09\x0b\x67\x1b\x52'
code += b'\x41\x6b\x40\x5d\x56\x17\x5d\x01'
code += b'\x3a\x04\x13\x1d\x68\x50\x06\x45'

print("".join([chr(a ^ b) for a, b in zip(key, code)]))
```

The flag is `flag{netB5D_i5_4ws0m3_y0u_sh0uld_7ry_i7}`
