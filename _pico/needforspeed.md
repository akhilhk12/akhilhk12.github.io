---
layout: post
title:  "NeedForSpeed"
grouped_by: picorev
---
### Description
> The name of the game is speed. Are you quick enough to solve this problem and keep it above 50 mph? need-for-speed.

## Analysis

Opening in ghidra we find the following functions in main
```c
   header();
  set_timer();
  get_key();
  print_flag();

```
Looking more `set_timer` has a signal handler and alarm, so ifwe take too long, the program will stop.  
`get_key()` has a function that calculates the key

```c
  int local_c;
  
  local_c = -0x319e0722;
  do {
    local_c = local_c + -1;
  } while (local_c != -0x18cf0391);
  return 0xe730fc6f;
```
This just returns `0xe730fc6f` and the loop is to waste some time to make us hit the alarm.  

`print_flag()` validates if the key is as above. But not in time for the signal to block us out.



## Solution

Theres one of two ways I solved this. 

### Option 1 - the easier way

Since we know theres a signal_handler and an alarm, in gdb we can set to ignore SIGALRM and run the program. It should now not stop the program and give us the key.

```
$>chmod +x need-for-speed                                                                                                                                                                                                                                           
$> gdb ./need-for-speed                                                                                                                                                                                                                                              
GNU gdb (Ubuntu 9.1-0ubuntu1) 9.1
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
GEF for linux ready, type `gef' to start, `gef config' to configure
88 commands loaded and 5 functions added for GDB 9.1 in 0.00ms using Python engine 3.8
Reading symbols from ./need-for-speed...
(No debugging symbols found in ./need-for-speed)

gef➤  handle SIGALRM ignore
Signal        Stop      Print   Pass to program Description
SIGALRM       No        Yes     No              Alarm clock
gef➤  r
Starting program: /mnt/c/Users/hrakh/Downloads/need-for-speed
Keep this thing over 50 mph!
============================

Creating key...

Program received signal SIGALRM, Alarm clock.
Finished
Printing flag:
PICOCTF{Good job keeping bus #190ca38b speeding along!}
[Inferior 1 (process 6657) exited normally]
```

There's the flag `PICOCTF{Good job keeping bus #190ca38b speeding along!}`  

### Option 2 -Setting flag to the value discovered in Analysis

Run the file in gdb and set a break on main. We now need to figure out the offset where `key` is stored so we'll execute 

```c
gef➤  break main
Breakpoint 1 at 0x91e
gef➤  r
Starting program: /mnt/c/Users/hrakh/Downloads/need-for-speed

Breakpoint 1, 0x000055555540091e in main ()

gef➤  x/32i get_key
   0x55555540087d <get_key>:    push   rbp
   0x55555540087e <get_key+1>:  mov    rbp,rsp
   0x555555400881 <get_key+4>:  lea    rdi,[rip+0x1a0]        # 0x555555400a28
   0x555555400888 <get_key+11>: call   0x555555400610 <puts@plt>
   0x55555540088d <get_key+16>: mov    eax,0x0
   0x555555400892 <get_key+21>: call   0x5555554007f1 <calculate_key>
   0x555555400897 <get_key+26>: mov    DWORD PTR [rip+0x2007bf],eax        # 0x55555560105c <key>
   0x55555540089d <get_key+32>: lea    rdi,[rip+0x194]        # 0x555555400a38
   0x5555554008a4 <get_key+39>: call   0x555555400610 <puts@plt>
   0x5555554008a9 <get_key+44>: nop
   0x5555554008aa <get_key+45>: pop    rbp
   0x5555554008ab <get_key+46>: ret
   0x5555554008ac <print_flag>: push   rbp
   0x5555554008ad <print_flag+1>:       mov    rbp,rsp
   0x5555554008b0 <print_flag+4>:       lea    rdi,[rip+0x18a]        # 0x555555400a41
   0x5555554008b7 <print_flag+11>:      call   0x555555400610 <puts@plt>
   0x5555554008bc <print_flag+16>:      mov    eax,DWORD PTR [rip+0x20079a]        # 0x55555560105c <key>
   0x5555554008c2 <print_flag+22>:      mov    edi,eax
   0x5555554008c4 <print_flag+24>:      call   0x55555540076a <decrypt_flag>
   0x5555554008c9 <print_flag+29>:      lea    rdi,[rip+0x200750]        # 0x555555601020 <flag>
   0x5555554008d0 <print_flag+36>:      call   0x555555400610 <puts@plt>
   0x5555554008d5 <print_flag+41>:      nop
   0x5555554008d6 <print_flag+42>:      pop    rbp
   0x5555554008d7 <print_flag+43>:      ret
   0x5555554008d8 <header>:     push   rbp
   0x5555554008d9 <header+1>:   mov    rbp,rsp
   0x5555554008dc <header+4>:   sub    rsp,0x10
   0x5555554008e0 <header+8>:   lea    rdi,[rip+0x169]        # 0x555555400a50 <title>
   0x5555554008e7 <header+15>:  call   0x555555400610 <puts@plt>
   0x5555554008ec <header+20>:  mov    DWORD PTR [rbp-0x4],0x0
   0x5555554008f3 <header+27>:  jmp    0x555555400903 <header+43>
   0x5555554008f5 <header+29>:  mov    edi,0x3d

```

We understand now that `*0x55555560105c` should have the key value we found in `get_key()`. Assigning and running `print_flag` gives us the result

```c
gef➤  set *0x55555560105c=0xe730fc6f
gef➤  call(int) print_flag()
Printing flag:
PICOCTF{Good job keeping bus #190ca38b speeding along!}
$2 = 0x38
```
Getting again to the same flag `PICOCTF{Good job keeping bus #190ca38b speeding along!}`