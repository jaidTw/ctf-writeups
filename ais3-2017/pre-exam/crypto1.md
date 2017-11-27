## crypto1
### sol
程式給了flag做xor後的結果，因此一樣用xor inverse就可以拿到flag
```c
#include <stdio.h>
int main() {
    int flags[10] = { 964600246 , 1376627084 , 1208859320 , 1482862807 , 1326295511 , 1181531558 , 2003814564 };
    int output[20] = {};
    char flag[20] = "AIS3";
    for(int i = 0; i < 7; ++i) {
        output[i] = flags[i] ^ (flags[0] ^ *(int *)flag);
    }
    printf("%s", (char *)output);
}
```
