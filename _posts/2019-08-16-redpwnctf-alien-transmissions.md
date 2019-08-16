---
title: "RedpwnCTF: Alien Transmissions"
categories: ctf crypto
---

This was one of the higher scoring challenges I completed.

We're given the following description:

```txt
Brownie plugged his USB stick in, ready to take a look at the alien
transmissions he had received. However, when he opened the document, he
discovered that it was all encrypted! Brownie remembered overhearing Tux
bragging about a 38 character xor key, and realized that he must have encrypted
his document. Help Brownie find the key to decrypt the transmissions!

The transmission list is formatted as a list of strings delimited by ։. Here's
an example list: /Z։Tk8V։gF։ue3T։dae9#

Note: ։ is not the regular English colon.
```

The description pretty much gives it away. We're looking at a simple XOR cipher
with a 38 character key. The key insight here is that every 38th character will
be XORed with the same char. Combining all of these characters into a string
allows you to do frequency analysis and hopefully recover the char. Rinse and
repeat to get the whole key and decrypt the message.

In this case, we probably can't do frequency analysis on the whole text, since
they are alien transmissions and presumably random. However, we are given an
interesting character!

The `։` character is Unicode code point 1417, or 0x0589 in UTF-8. Let's assume
that the rest of the plaintext is all single byte UTF-8 (i.e. ascii). There's
two ways we can XOR such a plaintext. We can XOR over the bytes, or the code
points. Either way, we should see a high frequency of certain characters in our
38 strings.

Let's try reading the encrypted text as UTF-8 first:

```python
with open('encrypted.txt', 'r') as f:
    ct = f.read()

strs = [ct[i::38] for i in range(0, 38)]
print(list(map(ord, strs[0])))
```

Running it:

```sh
$ python sol.py
[65, 80, 29, 1519, 24, 32, 38, 77, 56, 34, 1519...
```

This is it! 1519 appears many times, and is the only value above 100. This must
be our colon character. It's trivial to recover the key character by XORing this
value with 0x0589. We can just look for the largest char and assume it is our
colon.

```
key = []
pt_pieces = []
for s in strs:
    high = max(map(ord, s))
    k = high ^ 0x589
    key.append(k)
    decrypted = [ord(code_point) ^ k for code_point in s]
    pt_pieces.append(decrypted)

pt = []
for j in range(0, len(pt_pieces[0])):
    for i in range(0, 38):
        if j >= len(pt_pieces[i]):
            break
        pt.append(pt_pieces[i][j])
pt = ''.join(map(chr, pt))

#print(pt)
print(''.join(map(chr, key)))
```

I actually spent a long time looking at the plaintext before realizing it was
just random garbage, and realizing the key was the flag!

```sh
$ python sol.py
flag{7ux'5_un6u3554bl3_x0r_k3y_973068}
```
