# Impossible Password

## Let's deep down in assembly

Let's do this little challenge real quick, first we unzip the .zip  
A little chmod to execute the freshly out .bin
Let's try it, and after let's just do a `strings file.bin`  
Just after we can see effectively a print and some data, if we do a ltrace we see that the code do a compare with *SuperSeKretKey* so let's redo a ltrace and type in this key
They now prompt us a seconde code and compare it with *PhM%tyUmR6tI^-wfn|E1* and it give us a new Password so apparently this is dynamic.  

So give the file to a disassembler, I personally use Binary Ninja online tool!  
We can see different things and in the start a function called "main", so i enter it and we have a lot of var, I mark down that var10 is my password "SuperSeKretKey", and i continue through my code with a printf, a strcmp, another strcmp and a *je* to a certain address, [je](http://faydoc.tripod.com/cpu/je.htm) means "jump if condition met" so we continue in the subroutine.  

We see in this sub a printf, a scanf a subroutine and then a compare. So the idea is, the subroutine generate the next password and compare to our code! And just after we see a jne and call a new function, if we jump into it and see a function that do a while and put a char each time, so it print our flag for sure !  

## So what stop us now

Basically as we see upper, the "generatePssword" function is call before the printFlag and so when the comparaison occurs, the password is newly generated and so we can't get it.  
So maybe by understanding how the pass is generated we'll struggle less  
Sadly I didn't find out how to do it online so imma use Ghidra let me install it, i'll be back in a few minutes.  

So once in, u'll go to __libc_start_main and right click to see the references, it launches a function, let's rename the function "main" and jump into it, after a bit of remap with what we got from Binary Ninja (which is way more pleasant to work with i think), our main look like this

```c
void main(void)

{
  int iVar1;
  char *__s2;
  undefined encryptedFlag;
  undefined local_47;
  undefined local_46;
  undefined local_45;
  undefined local_44;
  undefined local_43;
  undefined local_42;
  undefined local_41;
  undefined local_40;
  undefined local_3f;
  undefined local_3e;
  undefined local_3d;
  undefined local_3c;
  undefined local_3b;
  undefined local_3a;
  undefined local_39;
  undefined local_38;
  undefined local_37;
  undefined local_36;
  undefined local_35;
  char user_input [20];
  int local_14;
  char *firstPassword;
  
  firstPassword = "SuperSeKretKey";
  encryptedFlag = 0x41;
  local_47 = 0x5d;
  local_46 = 0x4b;
  local_45 = 0x72;
  local_44 = 0x3d;
  local_43 = 0x39;
  local_42 = 0x6b;
  local_41 = 0x30;
  local_40 = 0x3d;
  local_3f = 0x30;
  local_3e = 0x6f;
  local_3d = 0x30;
  local_3c = 0x3b;
  local_3b = 0x6b;
  local_3a = 0x31;
  local_39 = 0x3f;
  local_38 = 0x6b;
  local_37 = 0x38;
  local_36 = 0x31;
  local_35 = 0x74;
  printf("* ");
  __isoc99_scanf(&DAT_00400a82,user_input);
  printf("[%s]\n",user_input);
  local_14 = strcmp(user_input,firstPassword);
  if (local_14 != 0) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  printf("** ");
  __isoc99_scanf(&DAT_00400a82,user_input);
  __s2 = (char *)generatePassword(0x14);
  iVar1 = strcmp(user_input,__s2);
  if (iVar1 == 0) {
    printFlag(&encryptedFlag);
  }
  return;
}
```

After making the printFlag readable we can understand what is happening:

```c
void printFlag(byte *param_1)
{
  int index;
  byte *flag;
  
  index = 0;
  flag = param_1;
  while ((*flag != 9 && (index < 0x14))) {
    putchar((int)(char)(*flag ^ 9));
    flag = flag + 1;
    index = index + 1;
  }
  putchar(10);
  return;
}
```

My main intuition was, in the main we have a lot of var, they surely are text value array and so if we retranscribe them it should be alright, let's use the ghidra function convert and we've something that look like a list of char:

```c
  encryptedFlag = 'A';
  local_47 = ']';
  local_46 = 'K';
  local_45 = 'r';
  local_44 = '=';
  local_43 = '9';
  local_42 = 'k';
  local_41 = '0';
  local_40 = '=';
  local_3f = '0';
  local_3e = 'o';
  local_3d = '0';
  local_3c = ';';
  local_3b = 'k';
  local_3a = '1';
  local_39 = '?';
  local_38 = 'k';
  local_37 = '8';
  local_36 = '1';
  local_35 = 't';
```

Now we have everything we need to recreate this function !  
To do this imma just recreate it in C and here is the final result:

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char tab[20] = {'A',']','K','r','=','9','k','0','=','0','o','0',';','k','1','?','k','8','1','t'}, flag[50], xored;
    int xor_key = 9, index = 0;

    while (index < 20)
    {
        xored = tab[index] ^ xor_key;
        flag[index] = xored;
        index++;
    }

    flag[index] = '\0';
    printf("%s",flag);

    return 0;
}
```

This way this isn't hard to understand at all !  
Let's just `gcc breaker.c -Wall -o breaked` and launch breaked and we got it !! 
The key is **CENSORED**!!  
Well down!!  
