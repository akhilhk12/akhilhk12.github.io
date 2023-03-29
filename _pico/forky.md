---
layout: post
title:  "forky"
grouped_by: picorev
---
### Description
In this program, identify the last integer value that is passed as parameter to the function doNothing().

## Analysis

As alwys we start by decompiling the executable. We move into the main function.

```c
undefined4 main(undefined1 param_1)

{
  int *piVar1;
  
  piVar1 = (int *)mmap((void *)0x0,4,3,0x21,-1,0);
  *piVar1 = 1000000000;
  fork();
  fork();
  fork();
  fork();
  *piVar1 = *piVar1 + 0x499602d2;
  doNothing(*piVar1);
  return 0;
}

```
The challenge is to understand what value we are passsing to the `doNothing` function. So no point in looking into that.
Another thing to note is that this is a 32-bit lsb. this can be found using file command.

```c
[files] file vuln                                                                                                                                                                                                                                                             23:10:10
vuln: ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=836c8d5ecaad6d64f4a358cf73d060d0c5050e87, not stripped
```

## Solution

The coder forks 4 times, which would mean we'll end up creating 16 child processes. Assuming there's no race condition, all we now need to do is carry out simple math. `0x499602d2` gets added 16 times to `piVar1` so that would mean we'll need to do 1000000000 + int(0x499602d2 * 0x10) which would give us `-721750240`.  
This looks right as it is a 32 bit executable. The final flag would be `picoCTF{-721750240}`
