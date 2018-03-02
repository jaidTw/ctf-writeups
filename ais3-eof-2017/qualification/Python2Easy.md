## Python2Easy (150)
Credits to my teammates.

### solution
server在檢查讀寫檔案是只會禁止對`secret.py`讀寫，所以可以對`secret.pyc`讀寫達成目的
`.pyc`檔案中第5~8個byte為timestamp，若與`.py`的timestamp不同則會在reload時重新生成`.pyc`
所以可以先讀取``secret.pyc`的前10個byte得到timestamp，再執行以下`.py`檔
並將得到的`.pyc` timestamp改為與server上的`secret.pyc`相同，並重新寫到`secret.pyc`
此時因為`secret.pyc`與`server.py`的timestamp相同，reload時會直接load我們寫好的`.pyc`
key即變為`'123'`，可以通過`check`獲得flag
```python
key = '123'
```
