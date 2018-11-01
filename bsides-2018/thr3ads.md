## thr3ads (reverse, 400)

### Solution

In `main`, it will launch 5 threads to run each stage:

```c
    if ( pthread_create(&th1, 0LL, (void *(*)(void *))stage1, 0LL) ) {
        ...
    }
    if ( pthread_create(&th2, 0LL, (void *(*)(void *))stage2, &arg) ) {
        ...
    }
    if ( pthread_create(&th3, 0LL, (void *(*)(void *))stage3, 0LL) ) {
        ...
    }
    if ( pthread_create(&th4, 0LL, (void *(*)(void *))stage4, 0LL) ) {
        ...
    }
    if ( pthread_create(&th5, 0LL, (void *(*)(void *))stage5, 0LL) )
    {
        ...
    }
```

If all stages were qualified, the flag will be printed.

#### stage1

```c
signed __int64 __fastcall stage1(void *arg) {
  arch = getenv("ARCH");
  ...
  for ( i = 0; ; ++i ) {
    pos = i;
    if ( pos >= strlen(arch) )
      break;
    s1[i] = arch[i] ^ 0xA;
  }
  if ( !strcmp(s1, "B:m]kx=y") ) {
    ...
    printf("\x1B[32m\n [:)] Stage 1 Qualified\x1B[0m", v3);
  }
}
```

Set `ARCH` to `"B:m]kx=y" ^ 0xA`, which is `H0gWar7s` to pass this stage.


#### stage2

```c
signed __int64 __fastcall stage2(void *arg) {
  if ( *(_DWORD *)arg == *((char *)arg + 4) ) {
    printf("\x1B[32m\n [:)] Stage 2 Qualified\x1B[0m");
  }
}

int main(...) {
  ...
  arg = 'A';
  v8 = **(_BYTE **)(v6 + 8);
  if ( pthread_create(&th2, 0LL, (void *(*)(void *))stage2, &arg) )
}
```

This stage will pass if the argv[1] of the program is `'A'`.

#### stage3

```c
char *__fastcall stage3(void *a1) {
  printf("\x1B[35m\n Enter password for stage 3 :\x1B[33m");
  v1 = (unsigned __int64)fgets(s, 25, stdin);
  if ( v1 != -1 && strlen(s) == 24 ) {
    if ( !strcmp(s, "He_who_must_not_be_named") ) {
      printf("\x1B[32m [:)] Stage 3 Qualified\x1B[0m");
    }
  }
}
```

Input `He_who_must_not_be_named` to pass this stage.

#### stage4

```c
void *__fastcall stage4(void *a1) {
  s2[0] = 'm';
  s2[1] = 'C';
  s2[2] = 'D';
  s2[3] = 'D';
  s2[4] = 'S';
  s2[5] = '\0';
  fp = fopen(".hidden.txt", "r");
  if ( fp ) {
    fgets(hidden, 6, fp);
    if ( strlen(hidden) == 5 ) {
      v4 = 42;
      for ( i = 0; i <= 4; ++i )
        hidden[i] ^= v4;
      if ( !strcmp(hidden, s2) ) {
        printf("\x1B[32m\n [:)] Stage 4 Qualified\x1B[0m", s2);
      }
    }
  }
}
```

Can pass this stage if the content of `.hidden.txt` is `"mCDDS" ^ 42`, which is `Ginny`.

#### stage5

```c
void *__fastcall stage5(void *a1) {
  if ( getcwd(&buf, 0x400uLL) ) {
    if ( strlen(&buf) != 26 || buf[22] != 'a' ) {
      pthread_mutex_lock(&mutex);
      printf("\x1B[31m\n [:(] Stage 5 Failed\x1B[0m", 1024LL);
      pthread_mutex_unlock(&mutex);
    }
  }
}
```

We can pass this stage if the path length is 26 and the 22nd character of the path is `'a'`.


So, finally, move the binary to a path where length is 26 and 22nd chracter is 'a', then run the following script.

```sh
#!/bin/bash

echo "He_who_must_not_be_named" | env ARCH="H0gWar7s" ./chall A
```

```
$ ./chall.sh
 [:)] Stage 5 Qualified
 [:)] Stage 1 Qualified
 [:)] Stage 2 Qualified
 Enter password for stage 3 : [:)] Stage 3 Qualified
 [:)] Stage 4 Qualified
[-->] No of stages cleared : 5
 -------------------------------------------
Verifying........
 [:)]  You seem to have Wizard origins
---------------------------------------------------------------------------
 The Vault is open
 Your Flag is == flag{W1th_L0v3_Fr0m_R3x!}
```
