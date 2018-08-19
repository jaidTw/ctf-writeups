## weird
### solution
觀落陰題，題目給了一個執行檔，執行後會發現
```
./weird: relocation error: ./weird: symbol 44444444444444444 version GLIBC_2.2.5 not defined in file libc.so.6 with link time reference
```
由於是relocation error，因此先看看relocation table發生什麼事。
```
$ objdump -R weird

weird：     檔案格式 elf64-x86-64

DYNAMIC RELOCATION RECORDS
OFFSET           TYPE              VALUE
0000000000600ff8 R_X86_64_GLOB_DAT  __gmon_start__
0000000000601018 R_X86_64_JUMP_SLOT  444444@GLIBC_2.2.5
0000000000601020 R_X86_64_JUMP_SLOT  444444@GLIBC_2.2.5
0000000000601028 R_X86_64_JUMP_SLOT  4444@GLIBC_2.2.5
0000000000601030 R_X86_64_JUMP_SLOT  444444444444444@OPENSSL_1.0.0
0000000000601038 R_X86_64_JUMP_SLOT  44444444444444444@GLIBC_2.2.5
0000000000601040 R_X86_64_JUMP_SLOT  444444@GLIBC_2.2.5
0000000000601048 R_X86_64_JUMP_SLOT  4444444444444444444@OPENSSL_1.0.0
0000000000601050 R_X86_64_JUMP_SLOT  4444444444444444@GLIBC_2.4
```
所有的entry名稱都被改掉了，因此`dl_resove()`時失敗，因此必須試著修復這些entry。

首先長度理論上會和原本的函式相同，可以藉此先篩選出一部分的函式，接著可以利用動態分析的方式，藉由觀察call function時的暫存器，以及call之前前幾條指令設定了哪些暫存器來推測函式的參數，可以簡單的判別出參數型別是pointer(記憶體位址)還是scalar。

先用gdb開啟接著start，追到第一個call 44444444444444444@plt，這個很明顯應該是__libc_start_main，長度是17個字元，且dynamically linked的程式一定會有的entry，修改後程式也正常執行到另外一個call才爆炸，可以推測應該沒錯。

再來觀察一部分的反組譯
```
$ objdump -d -Mintel weird
  400916:       48 8d 85 f0 fe ff ff    lea    rax,[rbp-0x110]
  40091d:       ba 80 00 00 00          mov    edx,0x80
  400922:       be 00 00 00 00          mov    esi,0x0
  400927:       48 89 c7                mov    rdi,rax
  40092a:       e8 41 fd ff ff          call   400670 <444444@plt>
  40092f:       48 8d 85 70 ff ff ff    lea    rax,[rbp-0x90]
  400936:       ba 80 00 00 00          mov    edx,0x80
  40093b:       be 00 00 00 00          mov    esi,0x0
  400940:       48 89 c7                mov    rdi,rax
  400943:       e8 28 fd ff ff          call   400670 <444444@plt>
```
此處分別是`444444(rbp-0x110, 0, 0x80)`, `444444(rbp-0x90, 0, 0x80)`，6個字的函式中比較符合的大概就是memset了，分別對stack上兩塊0x80的空間進行歸0。修改完後程式執行也沒有出錯。

接下來從字串找一些線索，先找出幾個字串的位址
```
$ strings -tx weird
    ae4 decrypt error
    af2 dec len: %d
    aff data: %s
```
可以試圖從反組譯中找到cross reference
```
  4009f6:       bf e4 0a 40 00          mov    edi,0x400ae4
  4009fb:       e8 80 fc ff ff          call   400680 <4444@plt>
  400a00:       b8 ff ff ff ff          mov    eax,0xffffffff
  400a05:       eb 3f                   jmp    400a46 <__gmon_start__@plt+0x366>
  400a07:       48 8d 85 f0 fe ff ff    lea    rax,[rbp-0x110]
  400a0e:       48 89 c7                mov    rdi,rax
  400a11:       e8 9a fc ff ff          call   4006b0 <444444@plt>
  400a16:       48 89 c6                mov    rsi,rax
  400a19:       bf f2 0a 40 00          mov    edi,0x400af2
  400a1e:       b8 00 00 00 00          mov    eax,0x0
  400a23:       e8 38 fc ff ff          call   400660 <444444@plt>
  400a28:       48 8d 85 f0 fe ff ff    lea    rax,[rbp-0x110]
  400a2f:       48 89 c6                mov    rsi,rax
  400a32:       bf ff 0a 40 00          mov    edi,0x400aff
  400a37:       b8 00 00 00 00          mov    eax,0x0
  400a3c:       e8 1f fc ff ff          call   400660 <444444@plt>
```
ae4的部份是一個一般字串，做為4444的參數，因此4444應該就是puts了。
再來可以看到下面的兩個400660(444444)，這裡用到af2和aff，這兩者裡面有placeholder，因此應該是printf()。

此外這邊還有一個4006b0<444444@plt>，他的參數只有一個，是stack上某個連續記憶體空間，也只能猜是strlen()了，最後glibc還有一個長度16個字的，出現在function的尾部，附近會有一個比較，這也就只能猜是__stack_chk_fail()了，如此一來只剩兩個openssl的entry，分別是15和19字。

先試著篩19字的function
```
nm -D -g /usr/lib/libcrypto.so.1.0.0  | grep -E "^[0-9a-f]+ [T/t] \w{19}\$"
0000000000095f80 T AES_set_decrypt_key
0000000000095f70 T AES_set_encrypt_key
```
從字串來判斷應該是對某個地方進行decrypt，且從assembly看起來有三個參數
```
  400866:       48 8d 95 e0 fe ff ff    lea    rdx,[rbp-0x120]
  40086d:       48 8b 85 c0 fe ff ff    mov    rax,QWORD PTR [rbp-0x140]
  400874:       be 80 00 00 00          mov    esi,0x80
  400879:       48 89 c7                mov    rdi,rax
  40087c:       e8 3f fe ff ff          call   4006c0 <4444444444444444444@plt>
```
應該是`int AES_set_decrypt_key( const unsigned char *userKey, const int bits, AES_KEY *key)`，接著最後長度為15的function，經常搭配使用的應該是
```
void AES_cbc_encrypt(
        const unsigned char *in,
        unsigned char *out,
        const unsigned long length,
        const AES_KEY *key,
        unsigned char *ivec,
        const int enc);
```

如此一來就把所有entry都復原了，最後結果如下：
```
$ objdump -R weird-patched

weird-patched：     檔案格式 elf64-x86-64

DYNAMIC RELOCATION RECORDS
OFFSET           TYPE              VALUE
0000000000600ff8 R_X86_64_GLOB_DAT  __gmon_start__
0000000000601018 R_X86_64_JUMP_SLOT  printf@GLIBC_2.2.5
0000000000601020 R_X86_64_JUMP_SLOT  memset@GLIBC_2.2.5
0000000000601028 R_X86_64_JUMP_SLOT  puts@GLIBC_2.2.5
0000000000601030 R_X86_64_JUMP_SLOT  AES_cbc_encrypt@OPENSSL_1.0.0
0000000000601038 R_X86_64_JUMP_SLOT  __libc_start_main@GLIBC_2.2.5
0000000000601040 R_X86_64_JUMP_SLOT  strlen@GLIBC_2.2.5
0000000000601048 R_X86_64_JUMP_SLOT  AES_set_decrypt_key@OPENSSL_1.0.0
0000000000601050 R_X86_64_JUMP_SLOT  __stack_chk_fail@GLIBC_2.4

$ ./weird-patched
dec len: 42
data: d0nT_F*Ck_Wl+h_mY_5+RT4B_y0u_m0+h3r_f*ck3r
```

data的部份即為flag。
