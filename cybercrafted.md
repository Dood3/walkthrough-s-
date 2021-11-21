# [CyberCrafted (THM)](https://tryhackme.com/room/cybercrafted)
Creator: [madrinch](https://tryhackme.com/p/madrinch)

***ANSWERS 1 & 2:***

To get these answers, only a thorough portscan is of essence.

--------------------------------------------------------------------------------------------

***ANSWER 3:***

The third question in this room asks for subdomains. So let's add something to our hosts file because there's no subdomain without cname.  
In my case, I just went for "cyber.thm" and tried to open that in my browser. Somewhat useless in this case,  
but it forwarded me to "cybercrafted.thm", which I hadn't in my hosts file yet. 

![PIC_BROWSER_1](https://user-images.githubusercontent.com/93183445/142762125-937aa05d-b22e-43ba-a61b-6f71ff12786b.png)

After updating my hosts file, things already looked more fun:

![PIC_BROWSER_2](https://user-images.githubusercontent.com/93183445/142762134-2ee455c2-b7b3-4716-97f8-3c563b1f932b.png)

I wasn't able to look at the source code of the site, because the picture on it was too big.  
But with the help of curl I was able to. The outcome was not of any use. The next step was, according  
the the above mentioned question for subdomains, to go after these.. Gobuster to the rescue:
```
gobuster vhost -u cybercrafted.thm -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt | cut -d " " -f 2
```
Which gave me the answer(s) to question 3:
```
[nope].cybercrafted.thm
[nope].cybercrafted.thm
[nope].cybercrafted.thm
```
I also added these subdomains to my hosts file.  
One of which presenting a login page. 

--------------------------------------------------------------------------------------------

***ANSWER 4:***

To let feroxbuster doing its thing, I let it loose on the found subdomains in alphabetical order.  
Meanwhile, I thought it would be a good idea to capture a request from the login page with Burp and  
throw sqlmap at it. Which turned out useless. I'll spare you the depressing picture of sqlmap just not being able to dump anything..
 
![PIC_FEROX](https://user-images.githubusercontent.com/93183445/142762339-5a073028-6219-468c-8996-52302caef76c.png)

--------------------------------------------------------------------------------------------

***ANSWERS 5 & 6:***

Upon visiting the newly found "vulnerable" page and entering just 'hello', I gave sqlmap a second chance, with a captured request.

![PIC_SRC](https://user-images.githubusercontent.com/93183445/142762394-9ba6a145-85c8-4add-a61b-390b9eff87ee.png)

Which turned out to be a rather fertile endeavour. I worked my way through the available data and ended up  
dumping the stuff I needed to proceed:
```
sqlmap -r req.txt --dbms=MySQL -D webapp -T admin --dump --batch
```
![PIC_SQL](https://user-images.githubusercontent.com/93183445/142762421-317229dd-8d26-4bb3-ab08-0cd41009f8d2.png)

So I got a hash to crack after echoing it to a file:
```
john --w=/usr/share/wordlists/rockyou.txt hash
```
Which gave me:

![PIC_JOHN](https://user-images.githubusercontent.com/93183445/142762434-9aaf5ae5-1d74-4b64-a883-c095917bd7ba.png)

--------------------------------------------------------------------------------------------

***ANSWERS 7, 8:***

With these credentials in my pocket (or on my screen rather), I returned to the login page on which sqlmap failed before and I was able to login.  
Oh come on.. Who could resist that ¯\\_(ツ)_/¯ ?!?!

![PIC_ADMIN_1](https://user-images.githubusercontent.com/93183445/142762451-828a6625-1fd1-40ea-b9c0-34b3fd98cea4.png)

Looking through the filesystem this way, obviously had its perks:

![PIC_ADMIN_2](https://user-images.githubusercontent.com/93183445/142762461-92be783a-70b5-4c3a-b197-01e7c5941dee.png)

I could make very good use of this treasure and login via ssh:
```
ssh -i id_rsa xxultimatecreeperxx@10.10.103.167
```
![PIC_JOHN_1](https://user-images.githubusercontent.com/93183445/142762474-2b087433-0d08-4699-9f5b-39d8c7089c61.png)

Or not...(yet). There had to be, again, some cracking going on to make that happen:
```
python3 /usr/share/john/ssh2john.py id_rsa > id_rsa.hash
john --w=/usr/share/wordlists/rockyou.txt id_rsa.hash
```
So I was (finally) able to login, but in the home directory wasn't anthing useful to be found.  
Linpeas helped me with that and I was able to find the answers for question 7, 8.  
To answer question 9, I still somehow had to get to be the other user:

![PIC_CREEPER](https://user-images.githubusercontent.com/93183445/142762519-1aa468cd-849c-4ef3-b7f0-3c8b5842568d.png)

On the way to that, I found a note:
```
Just implemented a new plugin within the server so now non-premium Minecraft accounts can game too! :)
- cybercrafted

P.S
Will remove the whitelist soon.
```
--------------------------------------------------------------------------------------------

***ANSWER 9:***

Going through the filesystem with the information I got from running linpeas, I found the password to become the user named '*cybercrafted*':

![PIC_CYBER](https://user-images.githubusercontent.com/93183445/142762547-956b005d-a1bf-48c5-a0b3-18b1f204abea.png)

--------------------------------------------------------------------------------------------

***ANSWER 10:***

![PIC_ROOT_1](https://user-images.githubusercontent.com/93183445/142762576-34fb7c27-cb7b-4d37-b63e-91ce83e292ac.png)
  
I admit, this one got me really bad. I understood, that this program would be startet with superuser rights  
and I would have to spawn a new instance out of it somehow. I just wasn't able to get it right. I never used this  
program before. A little help was needed.. Thanks to the creator of this room, [madrinch](https://tryhackme.com/p/madrinch), by the way :) !!!  
As it turned out, I also had to restart the machine...  

In the end, a little fingerbreakdance was all that was needed and I finally found what I've been coming here to find.  
(a bit frustration, some headscratches, satisfaction, and most of all the creation of new knowlege and skills for the challenges to come)  
After executing the command from above with sudo, I had to enter a shortcut, and I was root...

![PIC_ROOT_FINAL](https://user-images.githubusercontent.com/93183445/142762592-137eaf47-101c-40af-a21e-fbb8f92673ab.png)

***Thanks :)***
