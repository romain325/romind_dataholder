# Illumination

Illumination is one of the easiest challenge from the Forensics part of HackTheBox so I decidied to start with this one !!  
So apparently, from the description the breach is from the version control usage.  

Aaaaaand as everybody did in the past, forgetting to get rid of a key, making another commit removing it, and forgetting what is the purpose of a VERSION CONTROL TOOL !!  

So basically let's access the inside of the .git folder and especially the logs file and get the refs/heads/master file which contains all the commits and their commit token

Once I've my token I can now see the revision made on this commit with 

```sh
git show <COMMIT-FLAG>
```

And booom the user token appears !! Let's just decode it (base64 you know) and we got our token !!  
