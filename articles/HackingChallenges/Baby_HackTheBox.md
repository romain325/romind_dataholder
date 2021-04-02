# Bby md

So this one is pretty simple, extract but not with your regular command as the unzip seems to be broke so i used 7zip for linux with the command `7z e Baby_RE.md`  
The file tells that it's an executable file so let's do this
with a quick strings we can see this: 

```sh
Dont run `strings` on this challenge, that is not the way!!!!
Insert key: 
abcde122313
```

so let's run it with this key and BOOM the key: **HTB{B4BY_R3V_TH4TS_EZ}**