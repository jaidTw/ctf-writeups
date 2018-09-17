## Puppetmatryoshka

### Solution

```
$ tar xvf puppetmatryoshka.tar.gz
$ file matryoshka
matryoshka: tcpdump capture file (little-endian) - version 2.4 (Ethernet, capture length 262144)
```

Inside the capture file, there are three kismet requests and several tcp requests with length.
![](https://i.imgur.com/19Foo4w.png)

All data of kismet requests start with `BZh`, which is the header of bz2 format. First extract these payload from the file, The first and the third request can be decompressed successfully, and we'll get an ext4 image, respectively, but they are empty. While the payload extracted from the second kismet request failed on decompression, the reason is unexpected end. After observing the order of these request, I guess the payload of the second kismet request should be followed by those tcp requests, then I extract the payload of those tcp requests, and concat them together.

After the concatenation, it can be successfully decompressed, 

```
$ cat kismet2.decode tcp1.decode tcp2.decode tcp3.decode tcp4.decode tcp5.decode tcp6.decode > output
$ file output
output: bzip2 compressed data, block size = 900k
$ 7z x output -oextracted
```

Take a look at the extracted filesystem, I see a file `2501` and a very large journal file, si first check the `2501` out.
```
$ cd extracted
$ file output~
output~: Linux rev 1.0 ext4 filesystem data, UUID=b72f5197-d45b-4079-b6e8-b9d1be583d67 (extents) (huge files)
$ 7z x output~ -ofs
$ cd fs
$ exa -laBLT2
drwx------ 4         - xxxx users 14  9月 14:33 .
.rw-r--r-- 1    21,988 xxxx users 11  9月  3:18 ├── 2501
drwx------ 2         - xxxx users 14  9月 14:14 ├── [SYS]
.rw-r--r-- 1 1,048,576 xxxx users 11  9月  3:16 │  └── Journal
drwx------ 2         - xxxx users 11  9月  3:16 ├── lost+found
.rw-r--r-- 1         0 xxxx users 11  9月  3:18 ├── Section6
.rw-r--r-- 1         0 xxxx users 11  9月  3:18 └── Section9
$ file 2501
2501: 7-zip archive data, version 0.3
$ 7z x 2501 -o2501.extracted
$ cd 2501.extracted
```

After extract `2501`, you'll get a file and its content is... WOW! base64 encoded. Decode it and extract again, get in then run a rg you will see the flag.

```
$ base64 -d 2501 > out
$ file out
out: OpenDocument Text
$ 7z x out -oout.extracted
$ cd out.extracted
$ rg -e ".*(SECT.*}).*" -r '$1'
META-INF/documentsignatures.xml
17:SECT{Pupp3t_M4st3r_h1d35_1n_Th3_w1r3}
```

### Flag
```
SECT{Pupp3t_M4st3r_h1d35_1n_Th3_w1r3}
```
