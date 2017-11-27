## web2
### sol
```php
...
$db = array(
    array("username" => "sena", "password" => "0e959146861158620914280512624073"),
);
...

if ($username == $row["username"] and md5($password) == $row["password"])
{
    $msg = "Successful login as $username. Here's your flag: ".$flag;
    $success = true;
    break;
}
```
從source可以看出hash是用`==`做比較，而PHP的`==`有個廣為人知的漏洞，只要string以`0e`開頭會被轉換為浮點數0進行比較，而DB當中sena正好是以0e開頭，隨便找個hash是0e開頭的string丟進password即可。
e.g.
```
username : sena
pass : QNKCDZO
```
