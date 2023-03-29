---
layout: post
title:  "seed=sPRING"
grouped_by: picobin
---
### Description
> The most revolutionary game is finally available: seed sPRiNG is open right now! seed_spring. Connect to it with nc jupiter.challenges.picoctf.org 34558.

## Analysis

Decompiling the executable via ghidra
```c
undefined4 main(undefined param_1)

{
  uint local_20;
  uint local_1c;
  uint local_18;
  int local_14;
  undefined *local_10;
  
  local_10 = &param_1;
  puts("");
  puts("");
  puts("                                                                             ");
  puts("                          #                mmmmm  mmmmm    \"    mm   m   mmm ");
  puts("  mmm    mmm    mmm    mmm#          mmm   #   \"# #   \"# mmm    #\"m  # m\"   \"");
  puts(" #   \"  #\"  #  #\"  #  #\" \"#         #   \"  #mmm#\" #mmmm\"   #    # #m # #   mm");
  puts(
      "  \"\"\"m  #\"\"\"\"  #\"\"\"\"  #   #          \"\"\"m  #      #   \"m   #    #  # # #    #"
      );
  puts(" \"mmm\"  \"#mm\"  \"#mm\"  \"#m##         \"mmm\"  #      #    \" mm#mm  #   ##  \"mmm\"" );
  puts("                                                                             ");
  puts("");
  puts("");
  puts("Welcome! The game is easy: you jump on a sPRiNG.");
  puts("How high will you fly?");
  puts("");
  fflush(_stdout);
  local_18 = time((time_t *)0x0);
  srand(local_18);
  local_14 = 1;
  while( true ) {
    if (0x1e < local_14) {
      puts("Congratulation! You\'ve won! Here is your flag:\n");
      fflush(_stdout);
      get_flag();
      fflush(_stdout);
      return 0;
    }
    printf("LEVEL (%d/30)\n",local_14);
    puts("");
    local_1c = rand();
    local_1c = local_1c & 0xf;
    printf("Guess the height: ");
    fflush(_stdout);
    __isoc99_scanf(&DAT_00010caa,&local_20);
    fflush(_stdin);
    if (local_1c != local_20) break;
    local_14 = local_14 + 1;
  }
  puts("WRONG! Sorry, better luck next time!");
  fflush(_stdout);
                    /* WARNING: Subroutine does not return */
  exit(-1);
}
```
Looking into the code, the program takes inputs from user for 30 levels and each time compares the input with a random generated number bitwise and'ed with `0xf`. If we pas all 30, we end up getting the flag.  
The bigest giveaway here is  the `srand` function. From the challenge name as well, we now understansd it deals with pseudo random number generator which we will exploit. We know that if we give a same seed to `srand`, we'll end up getting the same value everytime. And what is the seed here? Its current unix time.  



## Solution

Knowing the above information, we can write a small c code that can give us the input based on current time for 30 levels bitwise `&` it with `0xf` and pipe this input to the netcat server.

```c
#include<stdio.h>
#include<stdlib.h>
#include<time.h>

void main() {
    srand(time(0));
    for(int i = 0; i < 30; i++){
        printf("%d\n", rand() &  0xf);
    }
}
```

Copiling and piping the output 

```
gcc -g random.c -o rand                                  
[ctf] ./rand | nc jupiter.challenges.picoctf.org 34558                         



                          #                mmmmm  mmmmm    "    mm   m   mmm
  mmm    mmm    mmm    mmm#          mmm   #   "# #   "# mmm    #"m  # m"   "
 #   "  #"  #  #"  #  #" "#         #   "  #mmm#" #mmmm"   #    # #m # #   mm
  """m  #""""  #""""  #   #          """m  #      #   "m   #    #  # # #    #
 "mmm"  "#mm"  "#mm"  "#m##         "mmm"  #      #    " mm#mm  #   ##  "mmm"



Welcome! The game is easy: you jump on a sPRiNG.
How high will you fly?

LEVEL (1/30)

Guess the height: LEVEL (2/30)

Guess the height: LEVEL (3/30)

Guess the height: LEVEL (4/30)

Guess the height: LEVEL (5/30)

Guess the height: LEVEL (6/30)

Guess the height: LEVEL (7/30)

Guess the height: LEVEL (8/30)

Guess the height: LEVEL (9/30)

Guess the height: LEVEL (10/30)

Guess the height: LEVEL (11/30)

Guess the height: LEVEL (12/30)

Guess the height: LEVEL (13/30)

Guess the height: LEVEL (14/30)

Guess the height: LEVEL (15/30)

Guess the height: LEVEL (16/30)

Guess the height: LEVEL (17/30)

Guess the height: LEVEL (18/30)

Guess the height: LEVEL (19/30)

Guess the height: LEVEL (20/30)

Guess the height: LEVEL (21/30)

Guess the height: LEVEL (22/30)

Guess the height: LEVEL (23/30)

Guess the height: LEVEL (24/30)

Guess the height: LEVEL (25/30)

Guess the height: LEVEL (26/30)

Guess the height: LEVEL (27/30)

Guess the height: LEVEL (28/30)

Guess the height: LEVEL (29/30)

Guess the height: LEVEL (30/30)

Guess the height: Congratulation! You've won! Here is your flag:

picoCTF{pseudo_random_number_generator_not_so_random_81b0dd7e}
```

We get the flag `picoCTF{pseudo_random_number_generator_not_so_random_81b0dd7e}`