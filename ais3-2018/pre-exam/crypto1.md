## POW
### solution
暴力解即可

### script
```python
def brute(cipher):
    chars = string.ascii_letters + string.digits
    for guess in itertools.product(chars, repeat=5):
        if sha256(cipher + ''.join(guess)).hexdigest()[:6] == '000000':
            return cipher + ''.join(guess)
    print "not found"
```

### flag
```
AIS3{Spid3r mAn - H3L1O wOR1d PrO0F 0F WOrK}
```

