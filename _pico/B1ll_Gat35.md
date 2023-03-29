---
layout: post
title:  "B11l_Gat35"
grouped_by: picorev
---
### Description
> Can you reverse this Windows Binary?

## Analysis

We decompile this code in ghidra to understand the code.Some looking around revealed `FUN_00408040` to be the main function. Here is the deompiled code.

```c
void FUN_00408040(void)

{
  int iVar1;
  undefined4 uVar2;
  int iStack_78;
  int iStack_74;
  char cStack_6d;
  undefined auStack_6c [100];
  uint uStack_8;
  
  uStack_8 = DAT_0047b174 ^ (uint)&stack0xfffffffc;
  thunk_FUN_004083e0(s_Input_a_number_between_1_and_5_d_0047b06c);
  thunk_FUN_00408430(&DAT_0047b094,&iStack_78);
  iStack_74 = 1;
  for (; 9 < iStack_78; iStack_78 = iStack_78 / 10) {
    iStack_74 = iStack_74 + 1;
  }
  if (iStack_74 < 6) {
    thunk_FUN_004083e0(s_Initializing..._0047b0b4);
    thunk_FUN_00407ff0(iStack_78,iStack_74);
    do {
      iVar1 = __fgetchar();
    } while (iVar1 != 10);
    thunk_FUN_004083e0(s_Enter_the_correct_key_to_get_the_0047b0c8);
    uVar2 = ___acrt_iob_func(0);
    thunk_FUN_004157db(auStack_6c,100,uVar2);
    cStack_6d = thunk_FUN_00407f60(auStack_6c);
    if (cStack_6d == '\0') {
      thunk_FUN_004083e0(s_Incorrect_key._Try_again._0047b0f8);
    }
    else {
      thunk_FUN_004083e0(s_Correct_input._Printing_flag:_0047b114);
      thunk_FUN_00408010();
    }
  }
  else {
    thunk_FUN_004083e0(s_Number_too_big._Try_again._0047b098);
  }
  @__security_check_cookie@4();
  return;
}
```
It looks like `thunk_FUN_00407f60` is validating a key and if it passes, prints the flag. We need to find what the key is and we should get our flag.

## Solution

If we take a closer look into the `if` condition after `thunk_FUN_00407f60`, we can see that the assembly instruction is as such


```c
        0040810e 0f  b6  45  97   MOVZX      EAX ,byte ptr [EBP  + local_6d ]
        00408112 85  c0           TEST       EAX ,EAX
        00408114 75  0f           JNZ        LAB_00408125
        00408116 68  f8  b0       PUSH       s_Incorrect_key._Try_again._0047b0f8             = "Incorrect key. Try again.\n"
                 47  00


```
The TEST instruction sets the CF and OF flag. The JNZ will jump to `LAB_00408125` if the the zero flag passes. We can do a bit of patching by changing the `JNZ` instruction to `JNC` so that the jump happens irrespective of any code as the `CF` flag is always 0 . Lets go ahead and do that on ghidra. Update the patch instruction to JNC and you'll be able to see the change reflected on disassembled code as well. The if condition is now removed and it always prints the flag.  
We'll export this patched exe as `original file` from ghidra and run it locally.

```
$> .\win-exec-2.exe
Input a number between 1 and 5 digits: 4353
Initializing...
Enter the correct key to get the access codes: 4334
Correct input. Printing flag: PICOCTF{These are the access codes to the vault: 1063340}
```
There you have it. The flag is `PICOCTF{These are the access codes to the vault: 1063340}`