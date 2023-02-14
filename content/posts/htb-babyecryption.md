---
layout: post
title: "[HTB] BabyEcryption writeup"
slug: "babyenryption"
date: "2023-02-14"
comments: false
categories:
  - hacking
tags:
  - ctf
  - write up
  - hack the box
  - cryptography
  - encryption/decryption
---

This is my write up for [Baby Ecryption](https://app.hackthebox.com/challenges/babyencryption) challenge in hackthebox. This challenge was fairly easy and just tested our our scripting skill and logical thinking. 

# Initial analysis

In this cryptography challenge we are provided with two files namely, `chall.py` and `msg.enc`.
The content of these files are:

**chall.py**
```python chall.py
import string
from secret import MSG

def encryption(msg):
    ct = []
    for char in msg:
        ct.append((123 * char + 18) % 256)
    return bytes(ct)

ct = encryption(MSG)
f = open('./msg.enc','w')
f.write(ct.hex())
f.close()

```

**msg.enc**
```text msg.enc
6e0a9372ec49a3f6930ed8723f9df6f6720ed8d89dc4937222ec7214d89d1e0e352ce0aa6ec82bf622227bb70e7fb7352249b7d893c493d8539dec8fb7935d490e7f9d22ec89b7a322ec8fd80e7f8921
```
We can observe that the ```chall.py``` python takes some ```secret``` and then encrypts the message by multiplying ```123``` to each character, then adding ```18``` and finally taking the mod of the number with ```256```. 
Bytes of the whole array is returned and then finally ```hex()``` to form the contents of ```msg.enc```. So to obtain the flag for the challenge we just have to find the ```secret``` from the ```msg.enc``` by reversing the encryption algorithm.

Due to the presence of the modulus operator, it is hard to backtrack the algorithm and reverse the encryption algorithm. So instead we will try to match each ascii characters with its encrypted form to get the desired flag. Our approach will involve bruteforcing every character with the ```msg.enc``` and appending the correct character to an array. 

So here is our python code for finding the flag:


**decryption.py**
```python decryption.py
f = open('./msg.enc', 'r')
enc = bytes.fromhex(f.read())


msg = []
for c in enc:
    for i in range(32,127):
        if ((123 * i + 18) % 256)== c:
            msg.append(chr(i))
print(''.join(msg))
```

Since valid characters for the flag will start form ascii values 33 - 127. We are iterating over those values only. Our code has runtime of O(n^2) and could probably be optimised using a hash map but since the the encrypted message is not long, it does not really matter in our case.

Output from our program.
```bash
> python3 decryption.py
Th3 nucl34r w1ll 4rr1v3 0n fr1d4y.HTB{**************************}
```
> That's it.


