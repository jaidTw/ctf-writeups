## dec dec dec (reverse) (warmup)

### Solution

After decompilation and tidy up the result, it's clear that main will get the input from `argv[1]`, copy and pass it through three functions, finally compare with some encoded string. The first function is obviously base64, the second is rot13, but the third is really difficult to figure out, after a while my teammate found its uuencode, so we decode the encoded string and get the flag.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

char *flag_encoded = "@25-Q44E233=,>E-M34=,,$LS5VEQ45)M2S-),7-$/3T";

int main(int argc, char **argv) {
  if ( argc != 2 ) {
    puts("./dec_dec_dec flag_string_is_here ");
    exit(0);
  }
  char *input = (char *)malloc(strlen(argv[1]) + 1);
  strncpy(input, argv[1], strlen(argv[1]));
  char *flag = (char *)uuencode(rot13(base64(input)));
  if ( !strcmp(flag, flag_encoded) )
    puts("correct  :)");
  else
    puts("incorrect :(");
  return 0LL;
}

char *base64(char *str) {
  char table[65] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

  unsigned int len = strlen(str);
  char *new_buf = malloc(4 * len / 3 + 1);
  char *p = new_buf;
  int i;
  for ( i = 0; i < len - len % 3; i += 3 ) {
    unsigned int q = (str[i] << 16) + (str[i + 1] << 8) + str[i + 2]
    p[0] = table[(q >> 18) & 0x3F];
    p[1] = table[(q >> 12) & 0x3F];
    p[2] = table[(q >> 6)  & 0x3F];
    p[3] = table[str[i + 2] & 0x3F];
    p += 4;
  }
  if ( len % 3 == 1 ) {
    p[0] = table[((unsigned int)(str[i] << 16) >> 18) & 0x3F];
    p[1] = table[16 * str[i] & 0x3F];
    p[2] = '=';
    p[3] = '=';
    p += 4;
  }
  else if ( len % 3 == 2 ) {
    unsigned int q = (str[i] << 16) + (str[i + 1] << 8);
    p[0] = table[(q >> 18) & 0x3F];
    p[1] = table[(q >> 12) & 0x3F];
    p[2] = table[(q >> 6) & 0x3F];
    p[3] = '='
    p += 4;
  }
  *p = 0;
  return new_buf;
}

char rot13(const char *str) {
  char *s = (char *)str;
  char *new_str = malloc(strlen(str) + 1);
  char *p = new_str;
  while ( *s ) {
    if ( *s <= '@' || *s > 'Z' ) {
      if ( *s <= '`' || *s > 'z' )
        *p = *s;
      else
        *p = (*s - 'T') % 26 + 'a';
    }
    else {
      *p = (*s - '4') % 26 + 'A';
    }
    ++p;
    ++s;
  }
  *p = 0;
  return new_str;
}

char *uuencode(char *str) {
  char *new_str = malloc(4 * strlen(str) / 3 + 1);
  char *s = str;
  char *p = new_str;
  unsigned int i;
  for ( i = strlen(str); i > 45; i -= 45 )
  {
  	p[0] = 'M';
    p++;
    for ( int j = 0; j <= 44; j += 3 )
    {
      if ( s[0] >> 2 )
        p[0] = (s[0] >> 2) + 32;
      else
        p[0] = 32;
      if ( 16 * s[0] & 0x30 )
        p[1] = (16 * s[0] & 0x30) + 32 | (s[1] >> 4);
      else
        p[1] = 32 | (s[1] >> 4);
      if ( 4 * s[1] & 0x3C )
        p[2] = ((4 * s[1] & 0x3C) + 32) | (s[2] >> 6);
      else
        p[2] = 32 | (s[2] >> 6);
      if ( s[2] & 0x3F )
        p[3] = (s[2] & 0x3F) + 32;
      else
        p[3] = 32;
      s += 3;
      p += 4;
    }
  }
  if ( i )
    *p = (i & 0x3F) + 32;
  else
    *p = 32;
  p++;
  for (int j = 0; j < i; j += 3 )
  {
    if ( s[0] >> 2 )
      p[0] = (s[0] >> 2) + 32;
    else
      p[0] = 32;
    if ( 16 * s[0] & 0x30 )
      p[1] = ((16 * s[0] & 0x30) + 32) | (s[1] >> 4);
    else
      p[1] = 32 | (s[1] >> 4);
    if ( 4 * s[1] & 0x3C )
      p[2] = ((4 * s[1] & 0x3C) + 32) | (s[2] >> 6);
    else
      p[2] = 32 | (s[2] >> 6);
    if ( s[2] & 0x3F )
      p[3] = (s[2] & 0x3F) + 32;
    else
      p[3] = 32;
    s += 3;
    p += 4;
  }
  *p = 0;
  return new_str;
}
```

### Flag
```
TWCTF{base64_rot13_uu}
```
