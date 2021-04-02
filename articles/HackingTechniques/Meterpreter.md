# Meterpreter  

A [[Meterpreter]] is a [[MetaSploit]] tool used in [[Hacking]] ! (What a surprise)

## Why is it so useful  

Fair question !  
We do have a lot of different tools to travel around a file system and execute basic commands.  
But a [[Meterpreter]] is a really Stealthy and Discret tool as it writes nothing to the disk and exclusively runs on the memory.  
And the most important thing: It doesn't create any independent process ! Meterpreter injects itself into a compromised process and can jump through compromised process! This way you don't trigger antivirus software.  
Those particularity make the [[Meterpreter]] a really powerful and discret tool to interact with.  
Your footprints are really tiny compared to other tools but you still have the versatility to execute commands and stuffs without having to setup a new payload each time.  

-------------------------------------

## The "Client Side"  

To interact with the newly injected [[Meterpreter]] we use a reverse_tcp shell
If you're wondering what is a [Reverse Shell](/article/HackingTechniques/Reverse_Shell.md), have a look at this article that explain how it works!  
So the Interaction is pretty simple!  

-------------------------------------

## Data-Transmission Protocol  

The Protocol used is the Type-Length-Value Protocol.  
This is a really simple protocol, Really easy to parse as it is the data type, followed by the Value-Length and then our Value.  
The parsing is so really smooth and barely human readable (interpretable is a more appropriate word!)

-------------------------------------

## What can I execute with this  

Tbh the list is really long and contains lots of useful tools.  
To interact with your local machine just preset you're command by "l".  

    - You can upload files `upload <FileName>`  
    - Start a Shell `shell`  
    - Snap a WebCam `webcam_snap`  
    - Know the IDLE state/time `idletime`  
    - IpConfig `ipconfig`  
    - Runs a command `execute -f YOUR_FILE`  
    - Edit File `edit <FileName>`  
    - Download File `download <FileName>`  
    - Remove and Clear Logs `clearev`  

And here is the [CheatSheet](https://www.sans.org/security-resources/sec560/misc_tools_sheet_v1.pdf)
