# OOOOOPSSIIIIIIE

## Don't u get bored of the first phase ?

I'm not gonna explain all the time --> **NMAP!**
And amazing, SSH (not pwnbl) and Apache on 80!  

## Some web!

Apparently we can login but no direct link, so let's have a look with a dedicated tool, the one that I love and hate: BURP !!  
In the Target page, we list all pages (after a first request of course) and we can see */cdn-cgi/login/*
And yeah a web form!! Let's use the last Password that wwe get in the previous Box! aka `MEGACORP_4dm1n!!`  
And well down we're in with the login *admin*, not that hard !!!  

## We're not high enough

Apparently there is a Super Admin, and we're not that powerful, let's get those rights!
In the Account page, we notice we have an accessId and burps tells us that effectively there is a cookie with our Id and our role
Let's try to bruteforce the id then !  
In Burp, let's make a request in account, send it to the intruder in burp, go to positions and use only the id in the get method
In the payloads tab add a list of number in the options (1..100/200) and allow every redirections in the options **IN REAL LIFE DONT DO THAT ITS REALLY DANGEROUS**  
Let's start the attack and wait for our response.  

We notice that the length does change sometimes, let's keep an eye on that.  
One is particularly long, the id 30, And in his response we see: *superadmin* !!  
We got it :) with a session id as 86575

## We should'nt be here

Let's try to get in upload by changing the user Cookie by our superadmin userId
Yes we're in !! What a great new isn't it ??
We can now upload files which is a great way to become root and pwn this box !  
Let's get a php reverse shell online and inject right in !! Don't forget to set the userId cookie to 86575

So now that our reverse_shell.php is in the sys let's find it.  
We'll need a new tool for that!!  let's use [dirsearch](https://github.com/maurosoria/dirsearch)
by running `python3 dirsearch.py -u http://10.10.10.28 -e php` we're getting every php file !!

And there are php files under a folder called /uploads !! perfect we're here!  


## Time for some reverse Shell!!  

Listen to the specified port in on shell (nc -lvvnp 4444 for example)
And trigger the reverse shell with another shell!  
`curl http://10.10.10.28/uploads/reverse_shell.php`

Now that we've our shell let's have a look at the connection pages(var/www/html) and particularly the db.php
```php
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

Thank you robert we've your password :))  
Let's *su* to his place

## Some BugTracking

We notice with command `id` that robert is part of the bugtracker group, 
after a find for file dedicated to this group we find */usr/bin/bugtracker*
And with a strings or gdb command shows us that it uses a shell function pretty well known *cat*
The idea here is replacing the default behaviour of cat to something that make us access root.  
Add /tmp/ to the path and create a file called cat, and echo "/bin/sh" in it, add chmod +x and you're ready to go !!
Execute **/usr/bin/bugtracker** once again and boom, it promote u as root.  
Delete the fake cat in tmp and let's now have a look to the file called user.txt and the one under /root/root.txt
The user code is `**CENSORED**`
And the Root is `**CENSORED**`


## We have to finish the job

If we go already, we're gonna be baddly blocked on the next box !! (as i've been)
Under the root folder and the hidden *.config* folder, there is a filezilla folder  
This file contains informations really interesting !  
Such as a User and a Password !! Let's keep near us !!!  

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes" ?>
<FileZilla3>
    <RecentServers>
        <Server>
            <Host>10.10.10.46</Host>
            <Port>21</Port>
            <Protocol>0</Protocol>
            <Type>0</Type>
            <User>ftpuser</User>
            <Pass>mc@F1l3ZilL4</Pass>
            <Logontype>1</Logontype>
            <TimezoneOffset>0</TimezoneOffset>
            <PasvMode>MODE_DEFAULT</PasvMode>
            <MaximumMultipleConnections>0</MaximumMultipleConnections>
            <EncodingType>Auto</EncodingType>
            <BypassProxy>0</BypassProxy>
        </Server>
    </RecentServers>
</FileZilla3>
```