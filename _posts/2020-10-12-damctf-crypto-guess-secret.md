---
title: "DamCTF: crypto/guess-secret"
categories: ctf crypto
---

This challenge asks us to break a 'super secure and efficient communication
method'. It also gives us the server code, although this isn't really
necessary---the server itself tells us all we need to know.

The server explains in detail what it does. It accepts a string, and creates the
plaintext by concatenating it to the flag. The plaintext is then compressed
using the lossless deflate (zlib) algorithm. Lastly, the compressed data is
encrypted using AES\_CTR\_128. The resulting ciphertext is returned to us.

There are two key insights here. The first is that the deflate algorithm
eliminates duplicate strings. The second is that CTR mode reveals the exact
length of the plaintext (or compressed data, in this case). This leaks how
"compressible" the data is, since inputting `AAAAAAAA` should result in a
shorter ciphertext than a uniformly random string of equivalent length.

The server also helpfully tells us the format of the flag, which is
`dam{[0-9a-f]+}` with 32 hexadecimal chars. So all we have to do is start with
`dam{` and start guessing flag characters.

```python
import ast
from pwn import *

io = remote('chals.damctf.xyz', 30308)
# initial text
io.recv()

chars = '0123456789abcdef'
flag = 'dam{'

for i in range(32):
    min_l, min_c = 999, '.'
    for c in chars:
        # query text
        io.recv()
	# send partial flag guess
	io.sendline(flag + c)
	# compare ciphertext lengths
	l = len(ast.literal_eval(io.recv().decode('ascii')))
	if l < min_l:
	    min_l, min_c = l, c
    flag += min_c

print(flag + '}')
```

It turns out that this works exactly as you would expect. With each correct
guess, the length of the ciphertext remains constant at 65 bytes, while
incorrect guesses are exactly one larger.

Running it gives us the flag within a few minutes:

```sh
$ python sol.py
dam{9f64ee1d4a7d6d8fe9136c3e9a74fc76}
```
