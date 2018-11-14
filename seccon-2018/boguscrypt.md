## Boguscrypt (Crypto)

### Solution

Decompile the binary and get
```c
int __cdecl main(int argc, const char **argv, const char **envp) {
  int result; // eax
  int key; // [esp+18h] [ebp-207Ch]
  struct stat stat; // [esp+34h] [ebp-2060h]
  char encrypted_flag[512]; // [esp+8Ch] [ebp-2008h]
  char buf[2048]; // [esp+88Ch] [ebp-1808h]
  char v16[512]; // [esp+108Ch] [ebp-1008h]
  char hostname_rev[2048]; // [esp+188Ch] [ebp-808h]

  printf("Key?:");
  __isoc99_scanf("%s", key);
  int addr = 33554559; // 127.0.0.2
  struct hostent *host = gethostbyaddr(&addr, 4u, AF_INET);
  if ( host ) {
    const char *hostname = host->h_name;
    int hostname_len = strlen(hostname);
    for (int i = 0; i < hostname_len; ++i )
      hostname_rev[hostname_len - i - 1] = hostname[i];
    memset(encrypted_flag, 0, 0x800u);
    memset(buf, 0, 0x800u);
    memset(v16, 0, 0x800u);
    int fd = open("flag.txt.encrypted", O_RDWR);
    fstat(fd, &stat);
    size_t len = stat.st_size;
    read(fd, encrypted_flag, stat.st_size);
    close(fd);
    len = strlen(encrypted_flag);
    dec(encrypted_flag, buf, len, hostname_rev);
    fd = open("flag.txt", 66, 0600);
    write(fd, buf, len);
    close(fd);
    result = dec(buf, v16, 2048, "abc");
  }
  else {
    herror("gethostbyaddr");
    result = 1;
  }
  return result;
}

int __cdecl dec(char *s1, char *s2, int length, char *key) {
  int result; // eax

  int len = strlen(key);
  for (int i = 0; ; ++i ) {
    result = i;
    if ( i >= length )
      break;
    s2[i] = s1[i] ^ key[i % len];
  }
  return result;
}
```
The program will get the hostname of `127.0.0.2`, reverse the hostname then xor with flag to encrypted the flag, thus we have to know the hostname to decrypt the flag.

We're also given a pcap, analyze it and will found a strange string `cur10us4ndl0ngh0stn4m3` inside a DNS packet.
Try to using it as the hostname to decrypt the flag:
```python
#!/usr/bin/env python3

hostname = b"cur10us4ndl0ngh0stn4m3"

with open("./flag.txt.encrypted", "rb") as f:
   flag = f.read()

hostname = hostname[::-1]
l = len(hostname)
for i in range(len(flag)):
   print(chr(flag[i] ^ hostname[i % l]), end="")
```

The output is `SECCON{This flag is encoded by bogus routine}`.
