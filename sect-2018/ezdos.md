## Ez dos (rev)

### Solution

We are given a DOS .com file, it will output the flag if the license entered is correct.
Read the disassembly and you will get the constraint :

```
str = "1337SHELL"
input[:4] == str[:4]
input[5] == "-"
input[6] == 0x66 ^ str[5]
input[7] == 0x79 ^ str[6]
input[8] == 0x74 ^ str[7]
input[9] == 0x79 ^ str[8]

=> input == "1337-5115"
```

### Flag

```
SECT{K3YG3N_MU51C_R0CK5}
```
