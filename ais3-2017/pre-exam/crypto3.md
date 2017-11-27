### crypto3
#### sol
```php
if (isset($_POST["username"]) and isset($_POST["password"]))
{
    $username = (string)$_POST["username"];
    $password = (string)$_POST["password"];

    $h1 = sha1($username);
    $h2 = sha1($password);

    if ($username == $password)
    {
        $msg = "Your password can not be your username.";
    }
    else if ($h1 === $h2)
    {
        $msg = "Flag1: $flag1";

        if (strpos($username, "Snoopy_do_not_like_cats_hahahaha") !== false and
            strpos($password, "ddaa_is_PHD") !== false and
            startsWith($h1, "f00d"))
        {
            $msg .= "</br>";
            $msg .= "Flag2: $flag2";
        }
    }
    else
    {
        $msg = "Invalid password.";
    }
```
這次是使用===比較了，所以0e的招沒用，所以要真的找sha1的collision，而之前很多新聞都有報導過，http://shattered.io/ 就能找到兩個會collide的pdf，作為參數送POST就能成功看到flag1了
```python
import requests
import urllib2

user = urllib2.urlopen("http://shattered.io/static/shattered-1.pdf").read()[:500];
pass = urllib2.urlopen("http://shattered.io/static/shattered-2.pdf").read()[:500];
r = requests.post('https://quiz.ais3.org:32670/index.php', data={'username': user, 'password': pass});

print r.text
```
