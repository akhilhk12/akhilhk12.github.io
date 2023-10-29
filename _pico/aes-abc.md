---
layout: post
title:  "AES-ABC"
grouped_by: picocrypto
---

### Description
> AES-ECB is bad, so I rolled my own cipher block chaining mechanism - Addition Block Chaining! You can find the source here: aes-abc.py. The AES-ABC flag is body.enc.ppm

## Analysis

Opening up the body.ec.ppm in GIMP, we see a very encrypted image.
[![gimpimagebody](https://i.imgur.com/O5nUFnp.png)](https://i.imgur.com/O5nUFnp.png)

Looking through the aes-abc.py code, the `aes_abc_encrypt` function looks interesting.

```python
def aes_abc_encrypt(pt):
    cipher = AES.new(KEY, AES.MODE_ECB)
    ct = cipher.encrypt(pad(pt))

    blocks = [ct[i * BLOCK_SIZE:(i+1) * BLOCK_SIZE] for i in range(len(ct) / BLOCK_SIZE)]
    iv = os.urandom(16)
    blocks.insert(0, iv)
    
    for i in range(len(blocks) - 1):
        prev_blk = int(blocks[i].encode('hex'), 16)
        curr_blk = int(blocks[i+1].encode('hex'), 16)

        n_curr_blk = (prev_blk + curr_blk) % UMAX
        blocks[i+1] = to_bytes(n_curr_blk)

    ct_abc = "".join(blocks)
 
    return iv, ct_abc, ct
```
What this essentially does is encrypt each byte in ECB mode and then does an additional manipulation ` n_curr_blk = (prev_blk + curr_blk) % UMAX`. Each ECB encrypted block is added with the prev block and the sum is moded with `UMAX`(from the code , we see is 256^16). Essentially our first step is to write a decoding logic that does the opposite - .  
Also a thing to note is that even thogh the above step will only give us the ECB encoded format, that should be enough for us to figure out the flag (check [this](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#/media/File:Tux_ECB.png) out). 

## Solution

We'll write our decrypt function using most of the same program provided. We'll also switch the input to be body and output to flag.


```python
#!/usr/bin/env python
import math

BLOCK_SIZE = 16
UMAX = int(math.pow(256, BLOCK_SIZE))


def to_bytes(n):
    s = hex(n)
    s_n = s[2:]
    if 'L' in s_n:
        s_n = s_n.replace('L', '')
    if len(s_n) % 2 != 0:
        s_n = '0' + s_n
    decoded = s_n.decode('hex')

    pad = (len(decoded) % BLOCK_SIZE)
    if pad != 0: 
        decoded = "\0" * (BLOCK_SIZE - pad) + decoded
    return decoded


def remove_line(s):
    # returns the header line, and the rest of the file0
    return s[:s.index('\n') + 1], s[s.index('\n')+1:]


def parse_header_ppm(f):
    data = f.read()

    header = ""

    for i in range(3):
        header_i, data = remove_line(data)
        header += str(header_i)

    return header, data
        

def pad(pt):
    padding = BLOCK_SIZE - len(pt) % BLOCK_SIZE
    return pt + (chr(padding) * padding)


def aes_abc_encrypt(ct):
    blocks = [ct[i * BLOCK_SIZE:(i+1) * BLOCK_SIZE] for i in range(len(ct) / BLOCK_SIZE)]

    final_block =[]    
    for i in range(len(blocks) - 1):
        prev_blk = int(blocks[i].encode('hex'), 16)
        curr_blk = int(blocks[i+1].encode('hex'), 16)

        n_curr_blk = (UMAX - prev_blk + curr_blk) % UMAX
        final_block.append(to_bytes(n_curr_blk))

    ct_abc = "".join(final_block)
 
    return 0, ct_abc, ct


if __name__=="__main__":
    with open('body.enc.ppm', 'rb') as f:
        header, data = parse_header_ppm(f)
    
    iv, c_img, ct = aes_abc_encrypt(data)

    with open('flag.ppm', 'wb') as fw:
        fw.write(header)
        fw.write(c_img)

```
Opening the `flag.ppm` file in GIMP we see
[![gimp-flag](https://i.imgur.com/uF0jR4O.png)](https://i.imgur.com/uF0jR4O.png)
We get the flag `picoCTF{d0Nt_r0ll_yoUr_0wN_aES}`