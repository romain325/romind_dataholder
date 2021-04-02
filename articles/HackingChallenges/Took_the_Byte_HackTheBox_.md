# Took The Byte

Let's do a lil bit more of Forensics from HTB !! The last one was very funny in fact !!  
So this one is a password recovery apparently  

So from the title, I deducted that if a byte has been removed, maybe I could just use a XOR BruteForce to get back the byte I miss so much to get my password  

So after 1.3sec of research I get on CyberChef, added the file, added a XOR Bruteforce with the key "password" because this is the name of the file so I'm sure it'll appear !  
And with the key ff, the word password.txt appears so I'm on the right way ! I just have to xor it with ff and I get the password !!!  

So now let's download the output of CyberChef and read the file inside !  And boom we've our password, sweeeeeeeeet !! 
