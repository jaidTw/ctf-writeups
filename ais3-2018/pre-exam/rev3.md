## crackme
### solution
題目是`.NET` binary，丟進IDA只能還原回CIL，可以用dotPeek這個`.NET` decompiler。丟進去後找到`Button1_Click()`這個函式，可以馬上看到一個array
```csharp=
    public MainWindow()
    {
      this.cnt = (object) 0;
      this.error_log = (object) "";
      this.secret = (object) new int[29]
      {
        234,
        226,
        248,
        152,
        208,
        154,
        223,
        244,
        226,
        158,
        244,
        238,
        234,
        216,
        210,
        244,
        223,
        228,
        244,
        232,
        249,
        159,
        200,
        192,
        244,
        230,
        206,
        138,
        214
      };
      this.InitializeComponent();
    }
```
接著下面的部分
```csharp
      object obj3 = Operators.AddObject(^(local4 = ref this.cnt), (object) 1);
      local4 = obj3;
      if (Operators.ConditionalCompareObjectEqual(this.cnt, (object) (int) ushort.MaxValue, false))
      {
        int num1 = checked (integer - 1);
        int num2 = 0;
        while (num2 <= num1)
        {
          // ISSUE: variable of a reference type
          object& local5;
          // ISSUE: explicit reference operation
          object obj4 = Operators.ConcatenateObject(^(local5 = ref this.error_log), (object) Convert.ToChar(Convert.ToInt32(RuntimeHelpers.GetObjectValue(NewLateBinding.LateIndexGet(this.secret, new object[1]
          {
            (object) num2
          }, (string[]) null))) ^ 171));
          local5 = obj4;
          checked { ++num2; }
        }
        Console.WriteLine(RuntimeHelpers.GetObjectValue(this.error_log));
      }
      int num3 = checked (integer - 1);
      int index2 = 0;
      while (index2 <= num3)
      {
        if (Operators.ConditionalCompareObjectNotEqual(NewLateBinding.LateIndexGet(this.secret, new object[1]
        {
          (object) index2
        }, (string[]) null), (object) (Convert.ToInt32(text[index2]) ^ 171), false))
          flag = false;
        checked { ++index2; }
      }
```
不是很懂在幹什麼，但是有兩行在做xor 171，總之試著把原本的secret拿去xor 171就能拿到FLAG了。

### flag
```
AIS3{1t_I5_EAsy_tO_CR4ck_Me!}
```
