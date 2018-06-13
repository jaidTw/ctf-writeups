## calc
### solution
這題的題目是Golang的binary，不過至少沒有strip，先跑跑看會發現有三個功能
1. Input key：讀取兩個key
2. Get Flag：顯示Your keys are wrong!!
3. Exit

總之先丟進IDA，從`main_main`開始看，Go的所有function前面都會有個`runtime_morestack_noctxt()`，這部分可以無視，根據其他function裡面`fmt_Println`的參數字串和數量可以大致還原出架構，參數跟變數都不重要。
```c
_BOOL8 __fastcall main_main(__int64 a1, __int64 a2, __int64 a3, __int64 a4, __int64 a5, __int64 a6) {
  show_banner();
  while ( !(_BYTE)result )
  {
    show_menu();
    show_prompt();
    read_int(a1);
    if ( choice == 1 )
    {
      inputkey();
    }
    else
    {
      if ( choice == 2 )
      {
        getflag();
        result = 0LL;
      }
      else
      {
        result = choice == 3;
      }
    }
  }
  return result;
}
```
`inputkey`裡面只是單純讀取字串，因此來看看`getflag`
```c
__int64 __fastcall getflag(__int64 a1, __int64 a2, __int64 a3, __int64 a4, __int64 a5, __int64 a6) {
  runtime_stringtoslicerune(a1, a2, a3, v6);
  v34 = v27;
  runtime_stringtoslicerune(a1, a2, v29, v27);
  op_Func3(a1, a2, v7, v8, v9, v10, v34, v28, v29, v27, v28);
  v31 = op_statictmp_0;
  v11 = duffcopy((char *)&v31 + 4, (char *)&op_statictmp_0 + 4);
  op_Func4((__int64)&v31 + 4, (__int64)&op_statictmp_0 + 4, v12, v13, v14, v15, v13, v12, v11, (__int64)&v31, 0x19uLL);
  if ( !result )
    return itface_Func6();
  itface_Func7();
  *(_QWORD *)&v16 = &unk_4C79D8;
  *((_QWORD *)&v16 + 1) = 8LL;
  io_ioutil_ReadFile((__int64)&v31 + 4, (__int64)&op_statictmp_0 + 4, v17, v18, v16);
  v33 = *(_QWORD *)v26;
  os_Exit((__int64)&v31 + 4);
  *(_QWORD *)&v19 = 0LL;
  *((_QWORD *)&v19 + 1) = v33;
  *(_OWORD *)v26 = *(_OWORD *)&v26[8];
  runtime_slicebytetostring(
    (__int64)&v31 + 4,
    (__int64)&op_statictmp_0 + 4,
    *(__int64 *)&v26[16],
    *(_DWORD *)v26,
    v20,
    v19);
  v35 = *(_QWORD *)&v26[16];
  v36 = 25LL;
  v37 = 0LL;
  runtime_convT2Estring((__int64)&v31 + 4, (__int64)&op_statictmp_0 + 4, v21, 25LL);
  v37 = *(_OWORD *)v26;
  return fmt_Println((__int64)&v31 + 4, (__int64)&op_statictmp_0 + 4, v22, *(__int64 *)&v26[16], v23, v24);
}
```
這邊可以先從第9行的`if`開始分割，下面有`io_ioutil_ReadFile`很可疑，繼續分析上面的`itface_Func7()`會發現裡面會輸出字串`Here you are (o^-’)b`，而`itface_Fun6`則是輸出`Your keys are wrong!!`所以推測只要if不成立就會讀檔拿到flag。

接著可以看到兩個`runtime_stringtoslicerune`，`op_Func3`和`op_Func4`，`runtime_stringtoslicerune`稍微查一下或用gdb trace，會發現把輸入的字串轉換成unicode runes，有兩次，因此應該是分別對key1和key2進行轉換。

接下來是分析op_Func3，裡面非常的複雜，光靜態分析也很難分析出變數的意義，因此試著用gdb去追蹤，觀察原本的key發生了什麼變化，這邊trace一陣子之後可以發現是把兩個unicode string進行相加(dword by dword)，之所以這麼複雜只是在判斷長度不同的部份, 確定功能以後這內容就不是那麼重要。

最後是`op_Func4`
```c
__int64 __fastcall op_Func4(__int64 a1, __int64 a2, __int64 a3, __int64 a4, __int64 a5, __int64 a6, __int64 a7, signed __int64 my_len, char a9, __int64 a10, unsigned __int64 secret_len) {

  result = my_len;
  if ( my_len == secret_len )
  {
    for ( i = 0LL; (signed __int64)i < my_len; ++i )
    {
      v13 = *(unsigned int *)(a7 + 4 * i);
      if ( i >= secret_len )
        runtime_panicindex(v13, i);
      if ( (_DWORD)v13 != *(_DWORD *)(a10 + 4 * i) )
        break;
    }
  }
  return result;
}
```
這邊推測是在驗證key是否正確，用gdb也可以看出是在跟記憶體中的某個unicode字串進行比較，所以只要使輸入的兩個key相加後等於這個字串即可，這個字串break在op_Func4的entry時會在`$rax`，可以直接用gdb dump出來。因為進去會先進行長度檢查，固定跟0x19比較，若不相等就直接return，可以得知這個字串長度應該為0x19，因此應該取以下位址的前19個字元。
```
0xc420059d8c:   0x0000006d0000fffd      0x0000000f0000fffd
0xc420059d9c:   0x000000580000006f      0x0000003100000020
0xc420059dac:   0x0000002000000073      0x0000004800000074
0xc420059dbc:   0x0000000000000030      0x000000530000fffd
0xc420059dcc:   0x0000002000000074      0x0000005400000063
0xc420059ddc:   0x0000002000000046      0x0000003a00000074
0xc420059dec:   0x000000310000006d      0x0000003300000032
0xc420059dfc:   0x0000003500000034      0x0000003700000036
0xc420059e0c:   0x0000006100000038      0x0000006300000062
0xc420059e1c:   0x0000006500000064      0x0000006700000066
```

比較trcik的部份就是裡面有包含2byte的字元，所以payload裡面應該用`"\uXXXX"`來encode，最後key為`"\ufffd\x6d\ufffd\x0f\x6f\x58\x20\x31\x73\x20\x74\x48\x30\x00\ufffd\x53\x74\x20\x63\x54\x46\x20\x74\x3a\x6d"`，讓他和`\x00`相加即可。


### script
```python
#!/usr/bin/env python3
from pwn import *
context(arch="amd64")

#r = process("calc")
r = remote("104.199.235.135", 2115)
a = "\ufffd\x6d\ufffd\x0f\x6f\x58\x20\x31\x73\x20\x74\x48\x30\x00\ufffd\x53\x74\x20\x63\x54\x46\x20\x74\x3a\x6d"
b = b"\x00"
r.recvuntil("Choice >")
r.sendline("1")
r.recvuntil("key >")
r.sendline(a)
r.recvuntil("key >")
r.sendline(b)
r.recvuntil("Choice >")
r.sendline("2")
r.interactive()
```
### flag
```
AIS3{G0_gO_g0_T0_h1Gh_!!!_R3v3rs3_oN_g0lauG_p1narY_1s_3xHanst3d_Orz}
```
