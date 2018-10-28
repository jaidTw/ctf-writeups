## st4tic (reverse, 200)

### Solution

The binary is a x64 ELF. First, try to decompile the program, but IDA can't identify where `main` is.

It's caused by some bytes were written to the fallthrough location of branches which will always taken, which obfuscates the decompiler.

![](https://i.imgur.com/UySxrdy.png)

After patching these bytes to `nop`, now the decompiler can correctly analyze all the functions, everything is clear.

```c
__int64 __fastcall main(int argc, char **argv)
{
  char *team_name; // [rsp+28h] [rbp-8h]

  team_name = getenv("team_name");
  if ( team_name && !strncmp(team_name, "bi0s", 4uLL) )
  {
    if ( argc == 2 )
    {
      if ( validate(team_name, argv[1]) == 1 )
        print_flag(argv[1]);
      else
        printf("Better luck next time!", argv);
    }
    else
    {
      printf("usage: chall <input>", "bi0s", argv);
    }
  }
  else
  {
    printf("Nope.", argv);
  }
  return 0LL;
}

int __fastcall validate(char *team_name, const char *flag)
{
  size_t _i; // rbx
  char c; // dl
  size_t len; // rax
  char buf[32]; // [rsp+10h] [rbp-60h]
  char encoded_flag[23]; // [rsp+30h] [rbp-40h]
  int v9; // [rsp+4Ch] [rbp-24h]
  int v10; // [rsp+50h] [rbp-20h]
  int j; // [rsp+54h] [rbp-1Ch]
  int ascii_sum; // [rsp+58h] [rbp-18h]
  int i; // [rsp+5Ch] [rbp-14h]

  ascii_sum = 0;
  j = 0;
  strcpy(encrypted_flag, "?5b9no=k!5<jW;h7W~b4#|");
  if ( strlen(flag) != 22 )
    return 0;
  for ( i = 0; ; ++i )
  {
    _i = i;
    if ( _i >= strlen(team_name) )
      break;
    ascii_sum += team_name[i];
  }
  v10 = 0;
  ascii_sum /= 30;
  while ( j != 22 )
  {
    if ( j & 1 )
      c = flag[j] - 4;
    else
      c = flag[j] + 4;
    buf[j] = c;
    buf[j] ^= ascii_sum;
    ++j;
  }
  v9 = 0;
  for ( i = strlen(buf) - 1; i >= 0; --i )
  {
    len = strlen(buf);
    if ( buf[len - i - 1] != encrypted_flag[i] )
      return 0;
  }
  return 1;
}
```

Only `bi0s` can pass the `team_name` check, it will be passed into `validate` to compute some values used to encrypt the flag(`argv[1]`). It's easy to write a inverse function.

```
#!/usr/bin/env python3

e = "?5b9no=k!5<jW;h7W~b4#|"
s = sum(b"bi0s") // 30

f = [None] * len(e)
e = e[::-1]

for i in range(len(e)):
    c = (ord(e[i]) ^ s) & 0xFF
    if i % 2 == 1:
        c += 4
    else:
        c -= 4
    f[i] = chr(c)

print("flag{%s}"% ("".join(f),))
```

flag : `flag{l34rn_7h3_b451c5_f1r57}`
