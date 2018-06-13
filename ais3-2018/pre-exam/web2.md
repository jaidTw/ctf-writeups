### hidden
#### solution
header中有`FLAG`等於`AIS3{NOT_A_VALID_FLAG}`，網頁會一直倒數然後可以redirect，因此寫一個script直到header改變時終止迴圈，跑到17332時會跳出迴圈。
#### flag
```
AIS3{g00d_u_know_how_2_script_4_W3B_7aad33de8e494a1a5765b1115db93cc3}
```
