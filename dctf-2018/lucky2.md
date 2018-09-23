## Even more lucky? (Exploit)

### Solution

Not quite a pwn challenge, reverse the binary first.

```cpp
__int64 __fastcall main(__int64 a1, char **a2, char **a3) {
  t = time(0);

  srand(t / 10);
  cout <<  "Hello, there!" << endl << endl << "What is your name?" << endl;
  cin.getline(buf);
  sub_2033((__int64)&v28, t / 10000, t / 10000, (unsigned int)t, v6, v7);
  serv_time = v28;
  cout << "I am glad to know you, " << buf << "!" << endl;
  cout << "Server time: " << serv_time << endl;
  cout << "If you guess the next 100 random numbers I shall give you the flag!" << endl << endl;
  for ( i = 0; (signed int)i <= 99; ++i )
  {
    num = rand();
    cout << "What number am I thinking of? [" << i << "/100]" << endl;
    cin >> buf;
    guess = stoi(&buf, 0LL, 10LL);
    if ( guess != num )
    {
      cout << "Wow that is wrong!" << endl;
      return -1;
    }
    cout << "Wow that is corect!" << endl << endl;
  }
  ifs = ifstream("./flag2")
  if ( is_open(ifs) )
  {
    ifs.getline(flag)
    cout << flag << endl;
    ifs.close()
  }
  return 0;
}
```

It will use `time(0)/10` as the seed for `srand()`, and output `time(0)/10000, so the probability to guess the seed at a given period is 1/1000. I wait for the server output time to be carried, and start brute-force guessing the time, then finally I will reach the correct server time, and get the flag.

### Exploit (if any)
```python
#!/usr/bin/env python3
from pwn import *
import subprocess

t = 153764029
for i in range(50):
    t += 1
    print("trying t =", t)
    r = remote("167.99.143.206", 65032)

    r.sendlineafter("name?", "123")
    data = subprocess.check_output(["./randgen", str(t)]).decode().split('\n')
    try:
        for i in range(100):
            recv = r.recvuntil("100]")
            print(recv)
            r.sendline(data[i])
        r.interactive()
    except EOFError:
        pass
```
```c
/* randgen.c */
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
    srand(atoi(argv[1]));
    for(int i = 0; i < 100; ++i) {
        printf("%d\n", rand());
    }
    return 0;
}
```

### Flag
```
DCTF{2e7aaa899a8b212ea6ebda3112d24559f2d2c540a9a29b1b47477ae8e5f20ace}
```
