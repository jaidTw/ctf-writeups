## A Tour of x86 - Part 1 50 (Reversing)

### Solution

In this chanllenge, we are asked several simple questions about the x86 assembly.

Q1 : What is the value of dh after line 129 executes? (Answer with a one-byte hex value, prefixed with '0x')
Ans : 0x00

```
129   xor dh, dh  ; <- Question 1
```
This simply clears the value of dh.


Q2 : What is the value of gs after line 145 executes? (Answer with a one-byte hex value, prefixed with '0x')
Ans : 0x00

```
131   mov dx, 0xffff  ; Hexadecimal
132   not dx
145   mov gs, dx ; to use them to help me clear     <- Question 2
```

`gs = ~0xffff = 0x00`

Q3 : What is the value of si after line 151 executes? (Answer with a two-byte hex value, prefixed with '0x')
Ans : 0x0000

```
107   mov cx, 0 ; The other two values get overwritten regardless, the value of ch and cl (the two components that make up cx) after this instruction are both 0, not 1.
149   mov sp, cx ; Stack Pointer
151   mov si, sp ; Source Index       <- Question 3
```

`si = sp = cx = 0`

Q4 : What is the value of ax after line 169 executes? (Answer with a two-byte hex value, prefixed with '0x')
Ans : 0x0e74

```
168   mov al, 't'
169   mov ah, 0x0e      ; <- question 4
```
The high 8 bit of ax is ah, and low 8 bit is al. The ascii value of 't' is 0x74, thus ax = 0x0e74

Q5 : What is the value of ax after line 199 executes for the first time? (Answer with a two-byte hex value, prefixed with '0x')
0x0e61

```
180     mov ax, .string_to_print
181     jmp print_string
182     .string_to_print: db "acOS", 0x0a, 0x0d, "  by Elyk", 0x00 
187   print_string:
188   .init:
189     mov si, ax
190
191   .print_char_loop:
192     cmp byte [si], 0
195     je .end
196
197    mov al, [si]  ; Since this is treated as a dereference of si, we are getting the BYTE AT si... `al = *si`
198
199    mov ah, 0x0e  ; <- Question 5!
200    int 0x10      ; Actually print the character
201
202    inc si        ; Increment the pointer, get to the next character
203    jmp .print_char_loop
204    .end:
```

At the first iteration, si will point to the start of string (character "a"), thus al = 0x61, and al = 0x0e61

### Flag

```
flag{rev_up_y0ur_3ng1nes_reeeeeeeeeeeeecruit5!}
```
