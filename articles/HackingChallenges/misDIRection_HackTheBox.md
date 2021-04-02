# MisDIRection

This challenge is one of the easiest from the HTB Misc challenges (yeah even if I like to solve things, I prefer the easiest one :))  

Anyway let's just start by unzipping with the code hackthebox and we got a bunch of file under the folder secret  
We have to find a malicious file in the middle of all this mess so it's gonna be pretty long if we do it all by hand  

So I'm just doing a `tree` into our secret folder to get some informations and just realized, we've the alphabet and numbers in some of them.  
Aaaaannd it does look weird because everything is empty so the key is hidden in this mess. What if the numbers just needs to be sorted and so you get like a code ??  
Let's try, in my .secret folder i run `find .` and grep the number out of them (only if they've one bc we don't care about the empty one) ! 
Alright then, now let's just sort them by this number, so we tell him the field separator is a / and then we want the 3rd part !  
And sorting is by numerical value (-n)  
Then I just grab the 2nd part of it (so the Letter), make it look like a single string, and as it look just like base64, I decode it! !!  

So here is my final command:  

```sh
find . | grep -E '\./.*/[0-9]+$' | sort -t/ -k3 -n | cut -d '/' -f2 | tr -d '\n' | base64 -d
```

And boom you got your key well down lad! 


