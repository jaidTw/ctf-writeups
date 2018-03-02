## simple (100)
### solution
首先用IDA對main進行反編譯會看到
```c
int main(int argc, const char **argv, const char **envp)
{
  v7 = 'orez';
  v8 = '\0';
  v5 = 'kcor';
  v6 = 'nam';
  f = XD;
  scanf("%s", &v4);
  if ( !strcmp(&v4, (const char *)&v7) )
  {
    f();
    atexit(XDDDD);
  }
  else
  {
    puts("Wrong password!");
  }
  return 0;
}
```
輸入字串會和`v7`比較，因此若是輸入`"zero"`則會呼叫`f()`(`XD()`)，並執行`atexit(XDDDD)`。
繼續反組譯`XD()`會發現給了一個錯誤的flag。
```c
int XD(void)
{
  return puts("flag{Submit_this_flag_and_fail_XD}");
}
```
接著反組譯`XDDDD()`
```c
void XDDDD(void)
{
  f = (int (*)(void))((char *)f + (char *)XDDD - (char *)XD);
  f();
}
```
由於`f`已經被設為`XD`，因此最後實際上是呼叫`XDDD()`
最後反組譯`XDDD()`
```c
int XDDD(void)
{

  ...
  GetEnvironmentVariableA("name", Buffer, 0x100u);
  result = strcmp(Buffer, rockman);
  if ( !result )
  {
    if ( IsDebuggerPresent() != 0 )
    {
      result = MessageBoxA(
                 0,
                 "Congratulations! Now you can have your flag: flag{Did you see the caption?}",
                 "Kidding",
                 0);
    }
    else
    {
      hModule = LoadLibraryA("kernel32.dll");
      GetProcAddress = (int (__stdcall *)(HMODULE, char *))::GetProcAddress(hModule, ProcName);
      hModule = LoadLibraryA("user32.dll");
      ...
      result = v33("notepad", 0);
      ...
    }
  }
  return result;
}
```
可以看到試圖從environment variable讀取`name`的值，並和`"rockman"`進行比較，成功後若是偵測到在使用debugger，則會再給出一個假的flag，若非則會執行else的內容，裡面比較值得注意的是有`"notepad"`，最後發現只要將notepad開啟，接著再新增一個環境變數`name`值為`"rockman"`，再執行程式輸入`"zero"`則會將flag寫到notepad中。
