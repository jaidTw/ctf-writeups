## simple_recovery 150 (Forensics)

### Solution

We are given two RAID5 disk images, and ... just extract them and run rg.

```
$ 7z x disk.img0.7z
$ 7z x disk.img1.7z
$ rg -a -e "flag\{"
disk.img0
265:                  <photoshop:LayerName>flag{dis_week_evry_week_dnt_be_securty_weak}</photoshop:LayerName>
266:                  <photoshop:LayerText>flag{dis_week_evry_week_dnt_be_securty_weak}</photoshopTOSHOPdOCUMENTaNCESTORy*RDFdESCRIPTION*RDFrdf*XXMPMETA****
```

### Flag

```
flag{dis_week_evry_week_dnt_be_securty_weak}
```
