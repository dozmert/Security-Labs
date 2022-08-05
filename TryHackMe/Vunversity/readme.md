## TryHackMe - Vulnversity
> Completed 28th April 2022, Revisited 4th May 2022.

As I found web applications to be a weak point in my knowledgebase, I decided to give this box as ago as a fun entry point. This box teaches active recon, web application attacks and privilege escalation. I did require some googling when it came to privilege escalation but once I understood what was needing to be done, I felt comfortable progressing to completion.

> Edit: I repeated this box in order to capture screenshots for the blog. In this second run through I filled in my process gaps and have highlighted where I changed my process.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nmap
- gobuster
- Buprsuite
- netcat

---
### Methodology
#### Objectives/Flags
1. User.txt
2. Root.txt

---
#### Reconnaissance
First i started off by running an nmap scan of the target machine to learn about its web service.
```bash
nmap -sV <Target-IP>
```
```bash
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))                 
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel  
```
From here we can see that the http server is running `Apache` on port `3333`. For the purposes of this box, we do not need to do any more passive recon and can move into actively enumerating the application.

This was the first time I've used `gobuster` and it ended up being very simple and easy to use. I used a publicly available wordlist that I found to be quite extensive and will be definitely coming back to use it for this type of purpose. The wordlist used was `httparchive_directories_1m_2021_04_28.txt` which can be found here: https://wordlists.assetnote.io/
```bash
gobuster dir -u http://<Target-IP>:3333 -w <wordlist>
```
As the wordlist was quite long it was going to take a long time to work through it. In hindsight I should have specified an output file that I could monitor as I found myself scrolling back through the history to comb through matches.

> Edit: In my second run through this box, I had appended `-o <outputfile.txt>` to my gobuster command to have the match results displayed neatly.

![](/TryHackMe/Vunversity/images/vulnversity_001.jpg)

```
//internal            (Status: 301) [Size: 324] [--> http://10.10.108.180:3333/internal/]
```
With the match found before the wordlist completed, I navigated to this sub directory in my browser and was met with a page with an upload form. To test out the form I uploaded a blank .txt file and received an error revealing that file extensions were being filtered.

![](/TryHackMe/Vunversity/images/vulnversity_002.jpg)

From here I moved to `Burpsuite` to make use of the intruder tool. I have used Burpsuite in the past however I have not had many chances to employ the intruder.
Once the proxy was set and I had set the web application within the scope of Burpsuite I reuploaded the blank .txt file to capture the traffic. This traffic was then sent to the intruder where I adjusted the appropriate settings to target the upload file's extension.

![](/TryHackMe/Vunversity/images/vulnversity_004.jpg)

This is where I found that I needed a new world list of file extensions. I found a publicly available list through Github which can be found here: https://gist.github.com/securifera/e7eed730cbe1ce43d0c29d7cd2d582f4
This list was quite extensive and was going to take some time to work through, for the purposes of this box I used the recommended list that was provided in the notes.
```txt
php
php2
php3
php4
phptml
```
This list failed a few times which had be questioning my intruder's settings however I discovered that I needed to uncheck the 'encode url' option to get successful results, I will keep this in mind in future. The results has shown the .phptml was not filtered and could successfully be uploaded to the server.

![](/TryHackMe/Vunversity/images/vulnversity_005.jpg)

Great, this was going to be the entry point. 

#### Weaponization 
I modified a php reverse shell script which can be found here: https://github.com/pentestmonkey/php-reverse-shell. This script was saved as a .phptml file.

#### Delivery
The modified script was uploaded to the server. At this point I prepared a netcat session to listened for the script once it was run.
```bash
nc -lvnp 1234
```

#### Exploitation
I have to disclose I missed a step here in by knowing how to execute the file. Through the boxes walkthrough a URL was disclosed that once navigated to would execute the script which would be picked up by netcat. If I return to this box at a later date, I'd like to continue with enumeration techniques to find this URL.

> Edit: On my second attempt at this box, I repeated the enumeration step to repeat the wordlist scan from the upload form's directory. I'm unsure if this was the right way to about it but if it works, it works.
```shell
gobuster dir -u http://<Target-IP>:3333/internal -w <Wordlist> -o <output.txt>
```

![](/TryHackMe/Vunversity/images/vulnversity_005.jpg)
![](/TryHackMe/Vunversity/images/vulnversity_006.jpg)

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
With shell access via netcat I was able to navigate to the user Bill's directory and find the `user.txt` flag.
```bash
ls /home/bill
  user.txt
    cat user.txt
```
At this point I was instructed to run a find command to SUID files on the system. This was the first time I have really interacted with SUID or GUID files so I had spent time reading into what this was to gain a rough idea as to why I was running the command. The documentation I was reading can be found here: https://www.thegeekdiary.com/linux-unix-how-to-find-files-which-has-suid-sgid-set/.
```bash
find / -user root -perm -4000 -exec ls -ldb {} \;
```
The 'interesting' results had been pointed out as `/bin/systemctl`. 

![](/TryHackMe/Vunversity/images/vulnversity_007.jpg)

With my previous Linux experience, I knew what systemctl was responsible on a surface level however how this was related to escalation in privileges was beyond my knowledge and experience. At this point I resorted to googling how this could be used in escalation of privileges. I found a useful resource which had explained that a custom service could be created and registered into the target machine allowing an execution of bash commands. The resource can be found here: https://medium.com/@klockw3rk/privilege-escalation-leveraging-misconfigured-systemctl-permissions-bc62b0b28d49
As I had no existing privileges on the machine to create a new file remotely, I knew this needed to be done my local machine and be uploaded to the server. I created a file called `rooot.service.phptml` and inserted in the lines of text which was provided by the resource.

![](/TryHackMe/Vunversity/images/vulnversity_008.jpg)

While writing this file I had that 'ah-hah' moment where I understood why systemctl was significant. Once the file was created and uploaded to the webserver I navigated to find in a file path I had learned through TryHackMe's web application course and knew that I simply needed to rename the file back into the `.service` format. 
```bash
cd /var/www/html/internal/uploads/rooot.service.phptml
mv rooot.service.phptml rooot.service
```
At this point the file was in its correct format and needed to be registered to the machine.
Using the web resource, I found the following command which I knew existed through previous linux exposure but never understood it's intention.
```bash
systemctl enable/var/www/html/internal/uploads/rooot.service
```

![](/TryHackMe/Vunversity/images/vulnversity_009.jpg)

Once the service was registered successfully, I was ready to execute the script. I had created a new Netcat listening on the local session prior to running the service command.
```bash
nc -lvnp 9999
```
Then proceed to start the malicious service through the shell.
```bash
systemctl start rooot.service
```

![](/TryHackMe/Vunversity/images/vulnversity_010.jpg)

This had now initiated a new shell session with root privileges. From here I was able to find the last flag in the root.txt file.
```bash
ls /root
  root.txt
    cat root.txt
```

---
### After-action
#### What I've learned
This was a fun learning experience in privilege escalation and allowed me get some brief experience with web application enumeration and attacking. I would have liked having some prior knowledge of systemctl's significance prior to attempting this box but I felt the exposure was a great learning point for me to carry forwards.

#### What I would've done differently
If I was to attempt this box again, I would strive to spend my time properly enumerating the box instead of relying on the walkthrough to guide me, this is mainly in regard to finding the URL to correctly execute the php reverse shell script once it was uploaded to the box.

> Edit: I am pleased with my second attempt at this box as I had filling in the gaps in process to the best of my ability.
