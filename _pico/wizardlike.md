---
layout: post
title:  "Wizardlike"
grouped_by: picorev
---
### Description
> Do you seek your destiny in these deplorable dungeons? If so, you may want to
look elsewhere. Many have gone before you and honestly, they've cleared out the
place of all monsters, ne'erdowells, bandits and every other sort of evil foe.
The dungeons themselves have seen better days too. There's a lot of missing
floors and key passages blocked off. You'd have to be a real wizard to make any
progress in this sorry excuse for a dungeon! Download the game. 'w', 'a', 's',
'd' moves your character and 'Q' quits. You'll need to improvise some wizardly
abilities to find the flag in this dungeon crawl. '.' is floor, '#' are walls,
'<' are stairs up to previous level, and '>' are stairs down to next level.

## Analysis

As a first step we decompile the iso provided within the challenge. Loading this in Ghidra, we get the entry point and move along to the main function.

Initial observation is that the game is creating the levels and 

```c
...
  while (bVar1) {
    if (DAT_00533f78 != DAT_00533f7c) {
      if (DAT_00533f7c == 1) {
        FUN_00401e05();
        uVar8 = FUN_00401e6d(0x51b840);
        DAT_00533f70 = 2;
        DAT_00533f74 = 1;
        DAT_00536790 = 0;
        DAT_00536794 = 0;
        DAT_00533f78 = 1;
      }
      else if (DAT_00533f7c == 2) {
        FUN_00401e05();
        uVar8 = FUN_00401e6d(0x51df60);
        DAT_00533f70 = 1;
        DAT_00533f74 = 2;
        DAT_00536790 = 0;
        DAT_00536794 = 0;
        DAT_00533f78 = 2;
      }
      else if (DAT_00533f7c == 3) {
        FUN_00401e05();
        uVar8 = FUN_00401e6d(0x520680);
        DAT_00533f70 = 2;
        DAT_00533f74 = 1;
        DAT_00536790 = 0;
        DAT_00536794 = 0;
        DAT_00533f78 = 3;
      }
...
```
Some further analysis reveals that the game although displays a 16X16 grid, has more characters in each line. This should be our first clue that the flag might be hiding in the ineccisible part(over the walls `"#"`).
Looking more into the code we see that once the user input is accepted, there are checks to call required  functions (up, down, left, right).

```c
...
    uVar4 = FUN_00403a00(extraout_XMM0_Qa_00,param_2,param_3,param_4,param_5,param_6,param_7,param _8
                         ,DAT_00536b50,param_10,extraout_RDX_02,pbVar5,(undefined *)param_13,
                         param_14);
    uVar8 = extraout_XMM0_Qa_01;
    if (uVar4 == 0x51) {
      bVar1 = false;
    }
    else if (uVar4 == 0x77) {
      uVar8 = FUN_00402247();
    }
    else if (uVar4 == 0x73) {
      uVar8 = FUN_004022c3();
    }
    else if (uVar4 == 0x61) {
      uVar8 = FUN_0040233f();
    }
    else if (uVar4 == 100) {
      uVar8 = FUN_004023bb();
    }
...
```

We now proceed to any of these functions. Lets look at `FUN_00402247()` which seems to be the __move up__ (as `0x77` is 'w').

```c
void FUN_00402247(void)

{
  undefined8 uVar1;
  
  uVar1 = FUN_00402188(DAT_00533f70,DAT_00533f74 + -1);
  if ((char)uVar1 != '\0') {
    if ((DAT_0053679c / 2 < DAT_00533f74) && (DAT_00533f74 <= 100 - DAT_0053679c / 2)) {
      DAT_00536794 = DAT_00536794 + -1;
    }
    DAT_00533f74 = DAT_00533f74 + -1;
  }
  return;
}

```
Seems like `uVar1` is the out of bounds check for our player. Diving more into `FUN_00402188`

```c
undefined8 FUN_00402188(int param_1,int param_2)

{
  undefined8 uVar1;
  
  if ((((param_1 < 100) && (param_2 < 100)) && (-1 < param_1)) && (-1 < param_2)) {
    if (((&DAT_0053a4a0)[(long)param_2 * 100 + (long)param_1] == '#') ||
       ((&DAT_0053a4a0)[(long)param_2 * 100 + (long)param_1] == ' ')) {
      uVar1 = 0;
    }
    else {
      uVar1 = 1;
    }
  }
  else {
    uVar1 = 0;
  }
  return uVar1;
}

```

The param1 and param2 seems to be the x and y cordinates of the player (current position) and the check returns a 0 when we hit `"#"` or a `" "`

We can now attempt to patch the game and help us move across the walls.

## Solution

We first look at the byte offset of `#` and `" "`in the binary. (Using ghidra itself)

```c
For `"#"`
        004021f4 3c  23           CMP        AL,0x23
        004021f6 74  3a           JZ         LAB_00402232

For `" "`
        0040222b 0f  b6  00       MOVZX      EAX ,byte ptr [RAX ]=>DAT_0053a4a0                = ??
        0040222e 3c  20           CMP        AL,0x20

```

We now update the hex value at offset `21f5` and `222f` to any value. I've changed it to `0x01`

[![gamehexeditor](https://i.imgur.com/Jpjqgm9.png)](https://i.imgur.com/Jpjqgm9.png)

Now running the patched game we are able to go beyond the walls. Here is the first level.

[![firstlevelgame](https://i.imgur.com/QnEQRwX.png)](https://i.imgur.com/QnEQRwX.png)

Going through the levels all 8 you'll be able to piece out the flag 
`picoCTF{ur_4_w1z4rd_8F4B04AE}`.

Be sure to scan the whole area.. Some of them are hidden way below or to the corners.