## MOSburger (200)
Credits to my teammates.

### solution
題目是VB6的bianry，直接反編譯無法獲得有用資訊。
試著用x64dbg調查，從字串著手發現有`"WRONG"`訊息，在該處下斷點，發現stack上有`"money$"`字串，試著輸入該字串，說給我些有用訊息就沒了，再調查string，發現有個字串`"mosburger.txt"`，但不知道在哪，再次下斷點，發現在appdata的資料夾中，裡面有mos code，上網decode變成binary code，再解碼get flag
