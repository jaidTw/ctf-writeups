## flags
### solution
這題顧名思義有很多flag，題目是一個JPG，圖片本身有一個假flag，`$ strings flags.jpg | grep "AIS3{"`可以再找到一個假flag，仔細觀察raw binary會發現圖片尾端有個zip檔，可以利用`binwalk -e flags.jpg`解出，裏面包含flag跟一個圖片Avengers_Infinity_War_Poster.jpg，大小為148837，丟去google可以發現[原圖](http://www.christinaperri.com/sites/g/files/g2000003451/f/Avengers_Infinity_War_poster.jpg)，以pkcrack就可以解開壓縮檔，但裡面的flag仍然是假的...

最後發現FLAG位於圖片中外框下方的摩斯電碼

這題真的令人很想罵幹...
### flag
```
ais3{youfindtherealflagohyeah}
```
