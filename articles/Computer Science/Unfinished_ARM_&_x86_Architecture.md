# ARM & x86 Architectures

As I've gained a lot of interest into Low Level programming and architecture, I faced the difference between ARM and x86 Architecture, but what are the differences between those too ?? What are they used to ?  

![Computer](https://cdn.systweak.com/content/wp/systweakblogsnew/uploads_new/2017/11/Blog-Image-Best-CPU-Benchmark-Software-For-Windows.jpg)
> Source:*Systweak.com*

## ARM Specs

![ARM](https://upload.wikimedia.org/wikipedia/commons/thumb/8/85/ARMSoCBlockDiagram.svg/460px-ARMSoCBlockDiagram.svg.png)

First, what does ARM stand for ??  
**A**dvanced **R**ISC **M**achine, ok great, but what does RISC stand for ?? **Reduced Instruction Set Computing**, ok sweet, but what does it mean ?  
Lots of questions ! Let's create a whole part about it!!  

### RISC

As I said earlier, RISC stand for Reduced Instruction Set Computing is based on a restrinct set of instructions which are highly optimized in opposition with the CISC, composed of complex instruction set.  

> Example: CISC is used by x86 but we'll see that later

There are multiple specificities to this Architecture, but the main features are:

- Large number of registers
- very regular instruction pipeline
- load/store arch (basically memory has his own specific instructions)

The large number of registers and the highly regular pipeline allow a low number of clock cycles per instruction.
And the load/store architecture is caracterised by his division between two categories: Memory Access(Interact between memory and registers) and ALU operations (Operations between registers)

This type of architecture is used in Low-end and mobile-systems, workstations, servers, and SuperCompters (Fugaku, the fastest Super Computer nowadays)

We won't go any further in this article, I'll maybe an article about that and link it !  

### Let's get Back to ARM

As ARM is based on RISC, ARM is effectively made for low-end and mobile-systems, we'll say that the usage are for Systems on Chips and Systems on Modules.  
As they are using RISC they require fewer components and it ends to be smaller which makes them cheaper, with less power consumption and better heat dissipation (really useful for smartphones and embbed systems and of course SuperComputers !)  

// TODO finish ARM ARch specs

## x86 Specs

// TODO Finish x86
CISC
Register Memory Arch

## Other Architectures

// TODO FINISH Other Architectures
- MIPS -- Ps,Ps2,N64,PsP
- RISC-V
- AVR -- Arduino, BMW
- SPARC -- Oracle
- PowerPC - GameCUbe,Wii,Ps3,Xbox360

- IBM System/360
- VAX