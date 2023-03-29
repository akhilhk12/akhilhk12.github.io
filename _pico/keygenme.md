---
layout: post
title:  "keygenme"
grouped_by: picorev
---
### Description
> Can you get the flag?
Reverse engineer this binary

## Analysis

Primary check is to see what kind of file this is. A simple file command yields 

```
keygenme: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=5ca9150c0fe861829a16033b7398e52e1e1b97af, for GNU/Linux 3.2.0, stripped
```
This is a 64bit LSB and it is stripped, which would mean we won't be using gdb.

We now decompile this executable in ghidra.Looking through the main function, we see that the user input is being processed and the result 0/1 is determining if our flag is right.
```c
  uVar1 = FUN_00101209(local_38);
  if ((char)uVar1 == '\0') {
    puts("That key is invalid.");
  }
  else {
    puts("That key is valid.");
  }
```
Lets look more into `FUN_00101209`. Decompiling this function gives

```c
  ...
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_98 = 0x7b4654436f636970;
  local_90 = 0x30795f676e317262;
  local_88 = 0x6b5f6e77305f7275;
  local_80 = 0x5f7933;
  local_ba = 0x7d;
  sVar1 = strlen((char *)&local_98);
  MD5((uchar *)&local_98,sVar1,local_b8);
  sVar1 = strlen((char *)&local_ba);
  MD5((uchar *)&local_ba,sVar1,local_a8);
  local_d0 = 0;
  for (local_cc = 0; local_cc < 0x10; local_cc = local_cc + 1) {
    sprintf(local_78 + local_d0,"%02x",(uint)local_b8[local_cc]);
    local_d0 = local_d0 + 2;
  }
  local_d0 = 0;
  for (local_c8 = 0; local_c8 < 0x10; local_c8 = local_c8 + 1) {
    sprintf(local_58 + local_d0,"%02x",(uint)local_a8[local_c8]);
    local_d0 = local_d0 + 2;
  }
  for (local_c4 = 0; local_c4 < 0x1b; local_c4 = local_c4 + 1) {
    acStack_38[local_c4] = *(char *)((long)&local_98 + (long)local_c4);
  }
  acStack_38[27] = local_6b;
  acStack_38[28] = local_66;
  acStack_38[29] = local_5b;
  acStack_38[30] = local_78[1];
  acStack_38[31] = local_6a;
  acStack_38[32] = local_60;
  acStack_38[33] = local_5e;
  acStack_38[34] = local_5b;
  acStack_38[35] = (undefined)local_ba;
  sVar1 = strlen(param_1);
  
  ...
```
We can immediately see that `local_98`, `local_90`  etc are Quad words containing the string put together `picoCTF{br1ng_y0ur_0wn_k3y_xxxxxx}`. The 8 characters ending the flag is what we need to figure out. Following in the decompiled code, we see `acStack[38]` is being assigned values that we can't figure out unless we let it run in a debugger. So we should be able to get the flag if we set a breakpoint after the stack is assigned.


## Solution

To run the logic of check, we need to give a sample input to the executable. For this, lets create a file with a sample flag

```
$> echo "picoCTF{br1ng_y0ur_0wn_k3y_xxxxxx}" > flag
```

I'll be now using `radare2` to debug this. Start r2 with the executable and then we'll provide the input and restart it.



```c
$> r2 -AA ./keygenme                                                                                                                                                                                                                                                  4:12:15
WARN: Relocs has not been applied. Please use `-e bin.relocs.apply=true` or `-e bin.cache=true` next time
INFO: Analyze all flags starting with sym. and entry0 (aa)
INFO: Analyze imports (af@@@i)
INFO: Analyze entrypoint (af@ entry0)
INFO: Analyze symbols (af@@@s)
INFO: Recovering variables
INFO: Analyze all functions arguments/locals (afva@@@F)
INFO: Analyze function calls (aac)
INFO: Analyze len bytes of instructions for references (aar)
INFO: Finding and parsing C++ vtables (avrr)
INFO: Analyzing methods
INFO: Recovering local variables (afva)
INFO: Type matching analysis for all functions (aaft)
INFO: Propagate noreturn information (aanr)
INFO: Scanning for strings constructed in code (/azs)
INFO: Finding function preludes (aap)
INFO: Enable anal.types.constraint for experimental type propagation
 -- Watch until the end!
[0x00001120]> dor stdin=flag
[0x00001120]> doo
INFO: File dbg:///mnt/c/Users/hrakh/Downloads/keygenme reopened in read-write mode
```
We'll set a breakpoint to main then print the function

```c
[0x00001120]> db main
[0x00001120]> dc
INFO: hit breakpoint at: 0x55cefeefe48b
[0x55cefeefe48b]> pdf
            ;-- rax:
            ;-- rip:
            ; DATA XREF from entry0 @ 0x55cefeefe141(r)
┌ 144: int main (int argc, char **argv, char **envp);
│           ; arg int argc @ rdi
│           ; arg char **argv @ rsi
│           ; var int64_t canary @ rbp-0x8
│           ; var char *s @ rbp-0x30
│           ; var int64_t var_34h @ rbp-0x34
│           ; var char **var_40h @ rbp-0x40
│           0x55cefeefe48b b    f30f1efa       endbr64
│           0x55cefeefe48f      55             push rbp
│           0x55cefeefe490      4889e5         mov rbp, rsp
│           0x55cefeefe493      4883ec40       sub rsp, 0x40
│           0x55cefeefe497      897dcc         mov dword [var_34h], edi ; argc
│           0x55cefeefe49a      488975c0       mov qword [var_40h], rsi ; argv
│           0x55cefeefe49e      64488b042528.  mov rax, qword fs:[0x28]
│           0x55cefeefe4a7      488945f8       mov qword [canary], rax
│           0x55cefeefe4ab      31c0           xor eax, eax
│           0x55cefeefe4ad      488d3d550b00.  lea rdi, str.Enter_your_license_key:_ ; 0x55cefeeff009 ; "Enter your license key: " ; const char *format
│           0x55cefeefe4b4      b800000000     mov eax, 0
│           0x55cefeefe4b9      e8f2fbffff     call sym.imp.printf     ; int printf(const char *format)
│           0x55cefeefe4be      488b154b2b00.  mov rdx, qword [reloc.stdin] ; [0x55cefef01010:8]=0x7fa505cb9980 ; FILE *stream
│           0x55cefeefe4c5      488d45d0       lea rax, [s]
│           0x55cefeefe4c9      be25000000     mov esi, 0x25           ; '%' ; 37 ; int size
│           0x55cefeefe4ce      4889c7         mov rdi, rax            ; char *s
│           0x55cefeefe4d1      e8fafbffff     call sym.imp.fgets      ; char *fgets(char *s, int size, FILE *stream)
│           0x55cefeefe4d6      488d45d0       lea rax, [s]
│           0x55cefeefe4da      4889c7         mov rdi, rax            ; char *arg1
│           0x55cefeefe4dd      e827fdffff     call fcn.00001209
│           0x55cefeefe4e2      84c0           test al, al
│       ┌─< 0x55cefeefe4e4      740e           je 0x55cefeefe4f4
│       │   0x55cefeefe4e6      488d3d350b00.  lea rdi, str.That_key_is_valid. ; 0x55cefeeff022 ; "That key is valid." ; const char *s
│       │   0x55cefeefe4ed      e8cefbffff     call sym.imp.puts       ; int puts(const char *s)
│      ┌──< 0x55cefeefe4f2      eb0c           jmp 0x55cefeefe500
│      ││   ; CODE XREF from main @ 0x55cefeefe4e4(x)
│      │└─> 0x55cefeefe4f4      488d3d3a0b00.  lea rdi, str.That_key_is_invalid. ; 0x55cefeeff035 ; "That key is invalid." ; const char *s
│      │    0x55cefeefe4fb      e8c0fbffff     call sym.imp.puts       ; int puts(const char *s)
│      │    ; CODE XREF from main @ 0x55cefeefe4f2(x)
│      └──> 0x55cefeefe500      b800000000     mov eax, 0
│           0x55cefeefe505      488b4df8       mov rcx, qword [canary]
│           0x55cefeefe509      6448330c2528.  xor rcx, qword fs:[0x28]
│       ┌─< 0x55cefeefe512      7405           je 0x55cefeefe519
│       │   0x55cefeefe514      e8f7fbffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
│       │   ; CODE XREF from main @ 0x55cefeefe512(x)
│       └─> 0x55cefeefe519      c9             leave
└           0x55cefeefe51a      c3             ret
[0x55cefeefe48b]>
```

We now look into `fcn.00001209` where our key is validated with

```
[0x55cefeefe48b]> pdf @fcn.00001209pdf 
```

```c
 ╎│   ; CODE XREF from fcn.00001209 @ 0x55cefeefe39a(x)
│      ╎└─> 0x55cefeefe3bf      83bd44ffffff.  cmp dword [var_bch], 0x1a
│      └──< 0x55cefeefe3c6      7ed4           jle 0x55cefeefe39c
│           0x55cefeefe3c8      0fb6459d       movzx eax, byte [var_63h]
│           0x55cefeefe3cc      8845eb         mov byte [var_15h], al
│           0x55cefeefe3cf      0fb645a2       movzx eax, byte [var_5eh]
│           0x55cefeefe3d3      8845ec         mov byte [var_14h], al
│           0x55cefeefe3d6      0fb645ad       movzx eax, byte [var_53h]
│           0x55cefeefe3da      8845ed         mov byte [var_13h], al
│           0x55cefeefe3dd      0fb64591       movzx eax, byte [var_6fh]
│           0x55cefeefe3e1      8845ee         mov byte [var_12h], al
│           0x55cefeefe3e4      0fb6459e       movzx eax, byte [var_62h]
│           0x55cefeefe3e8      8845ef         mov byte [var_11h], al
│           0x55cefeefe3eb      0fb645a8       movzx eax, byte [var_58h]
│           0x55cefeefe3ef      8845f0         mov byte [var_10h], al
│           0x55cefeefe3f2      0fb645aa       movzx eax, byte [var_56h]
│           0x55cefeefe3f6      8845f1         mov byte [var_fh], al
│           0x55cefeefe3f9      0fb645ad       movzx eax, byte [var_53h]
│           0x55cefeefe3fd      8845f2         mov byte [var_eh], al
│           0x55cefeefe400      0fb6854effff.  movzx eax, byte [var_b2h]
│           0x55cefeefe407      8845f3         mov byte [var_dh], al
│           0x55cefeefe40a      488b8528ffff.  mov rax, qword [var_d8h]
│           0x55cefeefe411      4889c7         mov rdi, rax            ; const char *s
│           0x55cefeefe414      e8c7fcffff     call sym.imp.strlen     ; size_t strlen(const char *s)
│           0x55cefeefe419      4883f824       cmp rax, 0x24           ; '$' ; 36
│       ┌─< 0x55cefeefe41d      7407           je 0x55cefeefe426
```

We find our necessary piece of code through `0x55cefeefe3c8` to ` 0x55cefeefe40a`. The instruction throughout uses `al` acumulation so all we now need to do is set a breakpoint to somewhere after the stack is moved. Lets set it to `0x55cefeefe40a`. Now to decide which `var` to check the flag for. We don't have to check each `var`, since the stack implementation is LIFO, we can just print `var_15h` and that should give us the rest of the flag. From the map below given while printing the function,

```c
; arg char *arg1 @ rdi
│           ; var int64_t canary @ rbp-0x8
│           ; var int64_t var_dh @ rbp-0xd
│           ; var int64_t var_eh @ rbp-0xe
│           ; var int64_t var_fh @ rbp-0xf
│           ; var int64_t var_10h @ rbp-0x10
│           ; var int64_t var_11h @ rbp-0x11
│           ; var int64_t var_12h @ rbp-0x12
│           ; var int64_t var_13h @ rbp-0x13
│           ; var int64_t var_14h @ rbp-0x14
│           ; var int64_t var_15h @ rbp-0x15
│           ; var int64_t var_30h @ rbp-0x30
│           ; var char *var_50h @ rbp-0x50
│           ; var int64_t var_53h @ rbp-0x53
│           ; var int64_t var_56h @ rbp-0x56
│           ; var int64_t var_58h @ rbp-0x58
│           ; var int64_t var_5eh @ rbp-0x5e
│           ; var int64_t var_62h @ rbp-0x62
│           ; var int64_t var_63h @ rbp-0x63
│           ; var int64_t var_6fh @ rbp-0x6f
│           ; var char *var_70h @ rbp-0x70
│           ; var int64_t var_78h @ rbp-0x78
│           ; var int64_t var_80h @ rbp-0x80
│           ; var int64_t var_88h @ rbp-0x88
│           ; var char *s @ rbp-0x90
│           ; var int64_t var_a0h @ rbp-0xa0
│           ; var int64_t var_b0h @ rbp-0xb0
│           ; var char *var_b2h @ rbp-0xb2
│           ; var signed int64_t var_b8h @ rbp-0xb8
│           ; var char *var_bch @ rbp-0xbc
│           ; var signed int64_t var_c0h @ rbp-0xc0
│           ; var signed int64_t var_c4h @ rbp-0xc4
│           ; var int64_t var_c8h @ rbp-0xc8
│           ; var char *var_d8h @ rbp-0xd8
```
 we see that `var_15h` is at `rbp-0x15`. So lets print the string using ps

 ```c
 [0x55cefeefe48b]> db 0x55cefeefe40a
[0x55cefeefe48b]> dc
INFO: hit breakpoint at: 0x55cefeefe40a
[0x55cefeefe40a]> ps @rbp-0x15
19836cd8}\xfd\x7f
[0x55cefeefe40a]>
 ```

So our flag is complete `picoCTF{br1ng_y0ur_0wn_k3y_19836cd8}`. Lets confirm this by running the executable and providing this flag as the input.

```
 $>./keygenme                                                                                                                                                                                       
Enter your license key: picoCTF{br1ng_y0ur_0wn_k3y_19836cd8}
That key is valid.
```

