## rev2
題目為一windows binary，先直接執行，發現有一個Text input，隨意輸入後點擊OK會出現`"Try harder!"`，以IDA Pro分析並藉由搜尋`Try harder`字串的xref位置找到主要的函式`DialogFunc()`，將其反編譯後並稍微改寫後得到下方的函式
```c
BOOL __stdcall DialogFunc(HWND hDlg, UINT a2, WPARAM nResult, LPARAM a4)
{
    char str[256] = {};
    GetDlgItemTextA(hDlg, 1001, &str, 256);
    // if len > 0, xor each byte with 0xCC
    if (strlen(str) > 0)
    {   
        for(uint i = 0; i < strlen(&str); ++i)
        {
            str[i] ^= 0xCCu;
        }
    }
    // compare xored input to unk_4120BC
    char *p1 = &str;
    char *p2 = &unk_4120BC;
    unsigned int len = 17;
    while ( *(_DWORD *)p1 == *(_DWORD *)p2 )
    {
        p1 += 4;
        p2 += 4;
        len -= 4;
        if ( len < 4)
        {
            if ( *p1 == *p2 )
            {
                MessageBoxA(0, "Congrats!", &Caption, 0x40u);
                return 1;
            }
            break;
        }
    }
    MessageBoxA(0, "Try harder!", &Caption, 0x30u);
    return 1;
}
```
分析後可以得知，當input與`0xCC`進行xor後，若是與`unk_4120BC`相同，則會跳出`"Congrats!"`，因此將`unk_4120BC`處字串與`0xCC`進行xor即可得到flag
