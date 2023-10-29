---
layout: post
title:  "VerySmooth"
grouped_by: picocrypto
---

### Description
> Forget safe primes... Here, we like to live life dangerously... >:)

## Analysis

Its good to note that with the hint provided this seems to point to pollard p-1 algorithm.
Looking into the gen.py code, we see that the value of e is `0x10001` and from the output.txt we get the values `n` and `c`. We can use [this](https://www.geeksforgeeks.org/pollard-p-1-algorithm/) implementation of p-1 alsorithm to get our factors p and q. Then further add to our code logic to decrypt.


## Solution

I wrote the following script including the pollard p-1 algorithm and an additional logic to decrypt the cyphertext.


```python
import math
from gmpy2 import *
import math
def pollard(n):
    a = 2
    i = 2
    while(True):
        a = pow(a,i,n)
        d = math.gcd((a-1), n)
        if (d > 1):  
            return d
            break
        i += 1
        
n = 0x4f7aa864f662a42a92220e372f5ff25a142aef26106a0dbdf573a66594966ac5dd03848745bb6a80402cad7ac6f2bf93f9ed840edd9c157dfd5d265ce2403e155a29666df8f9b98167ad2452e5a63fd0b7b14ffe966db60c6e2c65b0f602f5c22eb030c0335187759909abd4df622118c23463bcc42650e0a7761257452bf40069ca50dbe0c922d8823a9dcc4231b3952d31d1e977cb520528c6a450405f2a2ee6134db8c61ceb4478a647b0469712cc4f3d1369ef3dfd3d876a2c77bac5a149ccf3723a6e8c3ba1deb0675f25def8da9de2b3ac8b3e38d5ac5c9736b9af087b3fc53450136428e07d58fbc00f6609a4cc14eb0a13a7e76056a241256e03e95d
num = n
ans = []
while(True):
    d = pollard(num)
    ans.append(d)
    r = num//d
    ans.append(r)  
    break
c = 0xe41a61908eb48b85dc78975c288e62a271b1f237fdc958162727d2930b9af850e908137655c5955a078ff1aa63f5509fbaf79d179d24d209a061c36e0709437b8d2641f41d354bdea062084ea3be8637ed1c4bd8cf63d16c942976dd9d6188fc5e419afae17493d7cdb93d84052637d15e7fa1f852f4f5d786c86bfd024df0dfcf8431e7230cfbbce76a1835b178020ef839af42c377706918a50aac56f79285d743f4a177425eb00eaeb2bebe99343911ab653fe64bb61e140153b113f8554fe29561756fafc7460683d59dd3ee50eb48b718443b9f49e663b6dd02b0a15297468ec30a4f487e328103cdbc59d1d66fc4f03ef75ae45d6ce2035fdfaeb86b7
e = 0x10001

m = math.lcm(ans[0] - 1, ans[1] - 1)
d = pow(e, -1, m)
pt = bytes.fromhex(hex(pow(c,d,n))[2:])
print(pt.decode("ascii"))

```
```
\tmp> python solve.py
picoCTF{p0ll4rd_f4ct0r1z4at10n_FTW_7c8625a1}
```
We get the flag `picoCTF{p0ll4rd_f4ct0r1z4at10n_FTW_7c8625a1}`