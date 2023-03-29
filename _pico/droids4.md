---
layout: post
title:  "droids4"
grouped_by: picorev
---
### Description
>Reverse the pass, patch the file, get the flag.

## Analysis

Decompile the given apk using JADX. the decompilation yields a class `FlagstaffHill` with a `getFlag()` function


```c
package com.hellocmu.picoctf;

import android.content.Context;

/* loaded from: classes.dex */
public class FlagstaffHill {
    public static native String cardamom(String str);

    public static String getFlag(String input, Context ctx) {
        StringBuilder ace = new StringBuilder("aaa");
        StringBuilder jack = new StringBuilder("aaa");
        StringBuilder queen = new StringBuilder("aaa");
        StringBuilder king = new StringBuilder("aaa");
        ace.setCharAt(0, (char) (ace.charAt(0) + 4));
        ace.setCharAt(1, (char) (ace.charAt(1) + 19));
        ace.setCharAt(2, (char) (ace.charAt(2) + 18));
        jack.setCharAt(0, (char) (jack.charAt(0) + 7));
        jack.setCharAt(1, (char) (jack.charAt(1) + 0));
        jack.setCharAt(2, (char) (jack.charAt(2) + 1));
        queen.setCharAt(0, (char) (queen.charAt(0) + 0));
        queen.setCharAt(1, (char) (queen.charAt(1) + 11));
        queen.setCharAt(2, (char) (queen.charAt(2) + 15));
        king.setCharAt(0, (char) (king.charAt(0) + 14));
        king.setCharAt(1, (char) (king.charAt(1) + 20));
        king.setCharAt(2, (char) (king.charAt(2) + 15));
        String password = "".concat(queen.toString()).concat(jack.toString()).concat(ace.toString()).concat(king.toString());
        return input.equals(password) ? "call it" : "NOPE";
    }
}
```
We can run the java code on our end and get the password as `alphabetsoup`. No need to sit and decode the logic manually.

## Solution

From the previous challenge (`droids3`), we know that we can edit the `.smali` file and recompile to get our job done.

Decompile the apk using 

```
apktool d four.apk
```
Inside the decompiled folder, we will make changes to `four/smali/com/hellocmu/picoctf/FlagstaffHill.smali` file.
Replace the following code 

```c
    const-string v5, "call it"

    return-object v5
```
to

```c
    invoke-static {p0}, Lcom/hellocmu/picoctf/FlagstaffHill;->cardamom(Ljava/lang/String;)Ljava/lang/String;
     move-result-object v0
     return-object v0
```

This is similar to what we saw in `droids3`. We make use of `cardamom` as its inside the class `FlagstaffHill`. We move it to v0 and return it insted.  
Now recompile the code using 

```
$>apktool b four -o four_patched.apk
```
Also make sure to sign it before going ahead. For this run

```
$>keytool -genkeypair -v -keystore key.keystore -alias publishingdoc -keyalg RSA -keysize 2048 -validity 10000
```
This will ask you a passphrase. be sure to give in a min  6 character string. Remember this for signing later. Rest of the fields you can put empty and teh final question you can reply `yes`  
Then run 
```
$> jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore ./key.keystore four_patched.apk publishingdoc
```

You can now install this apk by dragging it into any AVD. Run the apk and put in the password `alphabetsoup` in the text field. This should yield you the flag

[![droids4](https://i.imgur.com/3I4R2Mb.png)](https://i.imgur.com/3I4R2Mb.png)

the flag is `picoCTF{not.particularly.silly}`.
