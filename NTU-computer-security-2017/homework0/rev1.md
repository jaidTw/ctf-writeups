## rev1
nm rev1可以看到一個function叫`print_flag`，反組譯後發現`main`只有`return 0`，因此用gdb開啟後`break main`再改變`$eip`跳到`print_flag`即可。
