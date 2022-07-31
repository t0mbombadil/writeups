AUTHOR: MADSTACKS

## Description
I wonder what this really is... enc `''.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)])`

## Solution

We actually need to see that from two 8 bytes characters this function made one 16 bytes character.
So bits shifts & masks help us extract those

```python
[ print( "%s%s" % (chr(ord(i) >> 8 ), (chr(ord(i) & 0b0000000011111111))), end='') for i in x ]
```

Bruteforce in cyberchef would do too.