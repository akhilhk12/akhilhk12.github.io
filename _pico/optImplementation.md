---
layout: post
title:  "otpImplementation"
grouped_by: picorev
---

### Description
> Yay reversing! Relevant files: otp flag.txt

## Analysis

Opening in ghidra we and analysing the main function,we see that the user input of a 100 characters is validated in `vaild_char` and then some operations are done to transform the input. And finally it is string compared with `"mlaebfkoibhoijfidblechbggcgldicegjbkcmolhdjihgmmieabohpdhjnciacbjjcnpcfaopigk pdfnoaknjlnlaohboimomb"` 
```c
      while( true ) {
      uVar4 = valid_char(local_e8[local_f0]);
      if ((int)uVar4 == 0) break;
      if (local_f0 == 0) {
        cVar2 = jumble(local_e8[0]);
        local_78[0] = cVar2 % '\x10';
      }
      else {
        cVar2 = jumble(local_e8[local_f0]);
        bVar1 = (byte)((int)cVar2 + (int)local_78[local_f0 + -1] >> 0x1f);
        local_78[local_f0] =
             ((char)((int)cVar2 + (int)local_78[local_f0 + -1]) + (bVar1 >> 4) & 0xf) - (bVar1 >> 4)
        ;
      }
      local_f0 = local_f0 + 1;
    }
    for (local_ec = 0; local_ec < local_f0; local_ec = local_ec + 1) {
      local_78[local_ec] = local_78[local_ec] + 'a';
    }
    if (local_f0 == 100) {
      iVar3 = strncmp(local_78,
                      "mlaebfkoibhoijfidblechbggcgldicegjbkcmolhdjihgmmieabohpdhjnciacbjjcnpcfaopigk pdfnoaknjlnlaohboimombk"
                      ,100);
      if (iVar3 == 0) {
        puts("You got the key, congrats! Now xor it with the flag!");
        uVar4 = 0;
        goto LAB_001009ea;
      }

```
Looking more into `valid_char`, we get to know the bounds of input used .

```c
undefined8 valid_char(char param_1)

{
  undefined8 uVar1;
  
  if ((param_1 < '0') || ('9' < param_1)) {
    if ((param_1 < 'a') || ('f' < param_1)) {
      uVar1 = 0;
    }
    else {
      uVar1 = 1;
    }
  }
  else {
    uVar1 = 1;
  }
  return uVar1;
}

```
So our key is a 100 cahracter string in the bounds of [0-9][a-f]. We do not have to decipher the logic since the end sring is given.  The length of char is not so high that we can brute force our way to the key.



## Solution

Lets run this on gdb and set a break at `strncmp` and check the values of `rsi` and `rdi` to see what we are comparing.

```c
gef➤  break strncmp@plt
Breakpoint 1 at 0x630
gef➤  r aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Starting program: /mnt/c/Users/hrakh/Downloads/otp aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa

Breakpoint 1, 0x0000555555400630 in strncmp@plt ()
[ Legend: Modified register | Code | Heap | Stack | String ]
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x00007fffffffdb00  →  "fkpejodinchmbglafkpejodinchmbglafkpejodinchmbglafk[...]"
$rbx   : 0x0000555555400a00  →  <__libc_csu_init+0> push r15
$rcx   : 0x60
$rdx   : 0x64
$rsp   : 0x00007fffffffda68  →  0x00005555554009c2  →  <main+436> test eax, eax
$rbp   : 0x00007fffffffdb70  →  0x0000000000000000
$rsi   : 0x0000555555400aa0  →  "mlaebfkoibhoijfidblechbggcgldicegjbkcmolhdjihgmmie[...]"
$rdi   : 0x00007fffffffdb00  →  "fkpejodinchmbglafkpejodinchmbglafkpejodinchmbglafk[...]"
$rip   : 0x0000555555400630  →  <strncmp@plt+0> jmp QWORD PTR [rip+0x200982]        # 0x555555600fb8 <strncmp@got.plt>
$r8    : 0x1a
$r9    : 0x00007ffff7fe0d60  →  <_dl_fini+0> endbr64
$r10   : 0x2b
$r11   : 0x2
$r12   : 0x0000555555400680  →  <_start+0> xor ebp, ebp
$r13   : 0x00007fffffffdc60  →  0x0000000000000002
$r14   : 0x0
$r15   : 0x0
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffda68│+0x0000: 0x00005555554009c2  →  <main+436> test eax, eax      ← $rsp
0x00007fffffffda70│+0x0008: 0x00007fffffffdc68  →  0x00007fffffffdeb5  →  "/mnt/c/Users/hrakh/Downloads/otp"
0x00007fffffffda78│+0x0010: 0x0000000200000340
0x00007fffffffda80│+0x0018: 0x0000034000000340
0x00007fffffffda88│+0x0020: 0x0000006400000064 ("d"?)
0x00007fffffffda90│+0x0028: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa[...]"
0x00007fffffffda98│+0x0030: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa[...]"
0x00007fffffffdaa0│+0x0038: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa[...]"
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x555555400620 <strncpy@plt+0>  jmp    QWORD PTR [rip+0x20098a]        # 0x555555600fb0 <strncpy@got.plt>
   0x555555400626 <strncpy@plt+6>  push   0x0
   0x55555540062b <strncpy@plt+11> jmp    0x555555400610
 → 0x555555400630 <strncmp@plt+0>  jmp    QWORD PTR [rip+0x200982]        # 0x555555600fb8 <strncmp@got.plt>
   0x555555400636 <strncmp@plt+6>  push   0x1
   0x55555540063b <strncmp@plt+11> jmp    0x555555400610
   0x555555400640 <puts@plt+0>     jmp    QWORD PTR [rip+0x20097a]        # 0x555555600fc0 <puts@got.plt>
   0x555555400646 <puts@plt+6>     push   0x2
   0x55555540064b <puts@plt+11>    jmp    0x555555400610
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "otp", stopped 0x555555400630 in strncmp@plt (), reason: BREAKPOINT
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x555555400630 → strncmp@plt()
[#1] 0x5555554009c2 → main()
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  x/s $rdi
0x7fffffffdb00: "fkpejodinchmbglafkpejodinchmbglafkpejodinchmbglafkpejodinchmbglafkpejodinchmbglafkpejodinchmbglafkpe\377\177"
gef➤  x/s $rsi
0x555555400aa0: "mlaebfkoibhoijfidblechbggcgldicegjbkcmolhdjihgmmieabohpdhjnciacbjjcnpcfaopigkpdfnoaknjlnlaohboimombk"
```
Seems like we are comparing the transformed input in `rdi` with what is expected.
Lets use pwn tools and spin up a python code and try to directly brute force the un-transformed result.

```c
from pwn import *

def findkey():
    bounds = "0123456789abcdef"
    finalkey =  ""
    toCompare = "mlaebfkoibhoijfidblechbggcgldicegjbkcmolhdjihgmmieabohpdhjnciacbjjcnpcfaopigkpdfnoaknjlnlaohboimombk"

    for l in range(100):
        for c in bounds:
            transformed = ""
            partialkey = finalkey + c + "a"*(99-l)
            p.sendline("r "+ partialkey)
            transformed = (p.recvuntil("(gdb)")).decode()
            if transformed[49+l] == toCompare[l]:
                finalkey += c
                print(finalkey)
                break
                
if __name__ == '__main__':
    p = process(["gdb", "./otp"])
    p.recvuntil("(gdb)")
    p.sendline("break *main+436")
    p.recvuntil("(gdb)")
    findkey()
```
In the above code I've used the offset 49+l because the decoding results in a string with a lot more information and the actual string used to compare starts from the 49th offset. Running the above script will yield you a 100 char string.
Mine was `6fa2e2a25c3b5869d75c7a5a062a4a51194c451e663fff306668ec42212a341f40cd199d78c72a21481596117a7c5e5217ac`
We can now XOR this with the content in flag.txt to get our flag `picoCTF{cust0m_jumbl3s_4r3nt_4_g0Od_1d3A_ca692500}`

