# **Narnia6 a new defi maybe?**
>PS: WHAT A DEFI, i really worked hard on this one

Even if we have a new defi, let's first read our code!

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char **argv){
        int i = 1;
        char buffer[64];

        snprintf(buffer, sizeof buffer, argv[1]);
        buffer[sizeof (buffer) - 1] = 0;
        printf("Change i's value from 1 -> 500. ");

        if(i==500){
                printf("GOOD\n");
        setreuid(geteuid(),geteuid());
                system("/bin/sh");
        }

        printf("No way...let me give you a hint!\n");
        printf("buffer : [%s] (%d)\n", buffer, strlen(buffer));
        printf ("i = %d (%p)\n", i, &i);
        return 0;
}
```

As always this bit of code provides us a lot of informations!!
Alright so apparently they are now preventing the kind of approach we were doing through here
SO we have to change the i value to 500, said like so it doesn't seems that hard, let's see
And as we deep down into a new type of attack, owasp and ressource like so are our best friends.
So after a bit of research I found a technique that matches pretty well what i want!
Here it is:
	- [Owasp Source](https://owasp.org/www-community/attacks/Format_string_attack)
	- [(Wikipedia Source)](https://en.wikipedia.org/wiki/Uncontrolled_format_string)
We can see that the fact they directly put argv[1] like this without using a "%s", argv[1] make this whole code vulnerable to this kind of attacks!

Apparently when running they provide us an hint so let's take it by running narnia5
```
###Here is our hint
buffer : [] (0)
i = 1 (0xffffd6b0)
```

**We do have a lot of informations, let's now jump into or exploitation**
--------------------------------------------------------------------------
So we do have the stack value given so now we do want to change it, but how?
First let's launch a gdb of narnia5 and test run it with some data
I'll want to see my hex dump at a time or another so i'll have to set a breakpoints somewhere
So let's use `disas main` and after a bit of reading we could find the point where my data is being pushed into this snprintf. So let's make a break point at this address: `break *0x08048528` (to find this adress just look at the right part of the disas and there is a "snprintf", that's him!)
So I was trying to find where the data i send to the program is stocked and it was goddamn long 
So I decided to focus on the tip! Let's use this address, we do want to post our "500" at this address
so let's just get or adress in little endian : \xe0\xd6\xff\xff
Ok so we need to declare it as his first arg so the CPU will be "oriented" to this address
let's just send a AAAA.%x.%x.%x.%x --> we can see which byte will be placed as first by our answer, and our aaaa is placed first so we'll have to precise our adress as first byte
Alright now, what's following next? Let's just send data to this address with some random %x just after, like so:
./narnia5 $(python -c 'print "\xe0\xd6\xff\xff" + "%11x%1$n"') --> MY I CHANGED?!?!?!?!
Alright before anything what is that %11x and this %1$n, %11x make a a hex sized of 11 bytes
AND NOW THE HARDEST TO UNDERSTAND (for me, honestly my brain was burning), So basically i just give him which byte he'll be in when we write it 1,2,3 or 4, so we want him at the first position!
perfect will be easier that i thought ;)) (honestly i was exhausted on this one and I was goddamn happy at this point)
Let's mess around with the number indicated to %x, if I put %20x, then my I = 24, well why?
I do have 4 byte of address and 20 of %x, so if i follow this logic, my %x should have %496x
Let's try this!
```bash
./narnia5 $(python -c 'print "\xe0\xd6\xff\xff" + "%496x%1$n"')
```

And PERFECT THE CMP DOES WORKED!!! so i've access to my new shell and i can use it to get narnia6 pass
~~**neezocaeng**~~

>PS: I spend a lot of time in the gdb, and be careful because, GDB provide us a loooot of data and sometimes your just on the wrong path! So take care of where you planned to go first, do have breaks and take time to be sure your on the right path

