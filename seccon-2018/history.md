## History (Forensics)

### Solution

```
$ unzip J.zip
$ binwalk J
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
3912330       0x3BB28A        ARJ archive data, header size: 22472, version 1, minimum version to extract: 1, compression method: stored, file type: binary, original name: "1", original file date: 1970-01-01 00:00:00, compressed file size: 538968064, uncompressed file size: 1441792, os: MS-DOS
```

Since MS-DOS stores data in 16bits little endian, append `-el` to `strings` for searching, let's try some keywords:
```
$ strings -el J | rg "SEC"
$ strings -el J | rg "CON"
```

And will find something like,
```
<CON{.txt
```

Then try
```
$ strings -el J | rg ".txt"
...
<SEC.txt
<SEC.txt
<SEC.txt
<SEC.txt
<CON{.txt
<CON{.txt
<CON{.txt
<F0r.txt
<F0r.txt
<tktksec.txt
<tktksec.txt
<tktksec.txt
<F0r.txt
<ensic.txt
<ensic.txt
<ensic.txt
<s.txt
<s.txt
<s.txt
<_usnjrnl.txt
<_usnjrnl.txt
<_usnjrnl.txt
<2018}.txt
<2018}.txt
```

The flag is `SECCON{F0rensics_usnjrnl2018}`
