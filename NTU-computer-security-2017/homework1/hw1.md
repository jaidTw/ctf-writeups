## hw1
以IDA Pro將題目反編譯，發現`encrypt`函式，大致整理後如下：
```c
int encrypt(char *str, int len)
{
  int v4; // edx@2
  int v5; // eax@2
  int ptr; // [sp+14h] [bp-20h]@2

  FILE *fp = fopen("flag", "wb");
  if ( len )
  {
    int i = 0;
    do
    {
      v4 = (i + 1) << (i + 2) % 0xAu;
      ptr = str[i++] * v4 + 9011;
      fwrite(&ptr, 4u, 1u, fp);
    }
    while ( i != len );
  }
  return fclose(fp);
}
```
可知加密公式為：
$$ c' = (c * ((i + 1) << (i + 2)\mod 0xAu))+9011$$
其中$c$為字元、$c'$為加密後之字元、$i$為字元index，
根據此段公式逆向計算即可解出FLAG
$$ c = \frac{c` - 9011}{(i + 1) << (i + 2) \mod 0xAu}$$
```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    FILE *fp = fopen("flag", "rb");

    int buf;
    int i = 0;
    while(fread(&buf, 4, 1, fp) == 1) {
        char c = (buf - 9011) / ((i + 1) << (i + 2) % 0xAu);
        putchar(c);
        i++;
    }
    return 0;
}

```
