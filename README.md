# Security-Labs
This is my offensive security blog/repo that’s very much a work in progress meant for students and beginners. Write-ups are nothing new however, I’m aiming to reinforce what I have learned by trying to explain it to others. I will mainly look to grasp on to fundamentals and industry tricks rather than show off fancy techniques. As this blog/repo matures it will look diffrent from when it first started.

## TryHackMe
I have based my lab documentation on a generalised intrusion kill-chain:
```
Reconnaissance > Weaponization > Delivery > Exploitation > Installation > Command & Control > Actions on Objective
```
### Practice Labs
| Activity | Difficulty | Description | Date Completed |
|---|:---:|:---|:---:|
| [Anthem](https://github.com/dozmert/Security-Labs/blob/main/TryHackMe/Anthem/readme.md) | Easy | Exploit a Windows machine in this beginner level challenge. | 25 MAR 2022 |
| [Blueprint](https://github.com/dozmert/Security-Labs/blob/main/TryHackMe/Blueprint/readme.md) | Easy | Hack into this Windows machine and escalate your privileges to Administrator. | 16 MAR 2022 |
| [Blue](https://github.com/dozmert/Security-Labs/tree/main/TryHackMe/Blue#readme) | Easy | Deploy & hack into a Windows machine, leveraging common misconfigurations issues. | 18 MAR 2022 |
| [Bookstore](https://github.com/dozmert/Security-Labs/tree/main/TryHackMe/Bookstore#readme) | Medium | A Beginner level box with basic web enumeration and REST API Fuzzing. | 07 JUN 2022 |
| [Dogcat](https://github.com/dozmert/Security-Labs/tree/main/TryHackMe/Dogcat#readme) | Medium | I made a website where you can look at pictures of dogs and/or cats! Exploit a PHP application via LFI and break out of a docker container. | 23 AUG 2022 |
| [Investigating Windows](https://github.com/dozmert/Security-Labs/tree/main/TryHackMe/Investigating-Windows#readme) | Easy | A windows machine has been hacked, its your job to go investigate this windows machine and find clues to what the hacker might have done. | 26 AUG 2022 | 
| [Kenobi](https://github.com/dozmert/Security-Labs/tree/main/TryHackMe/Kenobi#readme) | Easy | Walkthrough on exploiting a Linux machine. Enumerate Samba for shares, manipulate a vulnerable version of proftpd and escalate your privileges with path variable manipulation. | 03 AUG 2022 |
| [OhSINT](https://github.com/dozmert/Security-Labs/tree/main/TryHackMe/OhSINT#readme) | Easy | Are you able to use open source intelligence to solve this challenge? | 16 JUN 2022 | 
| [Overpass](https://github.com/dozmert/Security-Labs/tree/main/TryHackMe/Overpass#readme) | Easy | What happens when some broke CompSci students make a password manager? | In progress... | 
| [PickleRick](https://github.com/dozmert/Security-Labs/tree/main/TryHackMe/PickleRick#readme) | Easy | A Rick and Morty CTF. Help turn Rick back into a human! | 01 NOV 2021 | 
| [RootMe](https://github.com/dozmert/Security-Labs/tree/main/TryHackMe/RootMe#readme) | Easy | A ctf for beginners, can you root me? | 14 JUN 2022 | 
| [Searchlight](https://github.com/dozmert/Security-Labs/blob/main/TryHackMe/Searchlight/readme.md) | Easy | OSINT challenges in the imagery intelligence category | 30 JUN 2022 | 
| [SimpleCTF](https://github.com/dozmert/Security-Labs/blob/main/TryHackMe/SimpleCTF/readme.md) | Easy | Beginner level ctf | 28 JUN 2022 | 
| [Vulnversity](https://github.com/dozmert/Security-Labs/blob/main/TryHackMe/Vunversity/readme.md) | Easy | Learn about active recon, web app attacks and privilege escalation. | 28 APR 2022 |
| [Windows Fundamentals 1](https://github.com/dozmert/Security-Labs/blob/main/TryHackMe/Windows-Fundamentals-1/readme.md) | Info | In part 1 of the Windows Fundamentals module, we'll start our journey learning about the Windows desktop, the NTFS file system, UAC, the Control Panel, and more.. | In Progress... |
| [Wonderland](https://github.com/dozmert/Security-Labs/blob/main/TryHackMe/Wonderland/readme.md) | Medium | Fall down the rabbit hole and enter wonderland. | 05 AUG 2022 |

#### Fundamental Modules
| [Junior Penetration Tester](https://tryhackme.com/paths)
	[Introduction to Web Hacking](https://tryhackme.com/module/intro-to-web-hacking) - Get hands-on, learn about and exploit some of the most popular web application vulnerabilities seen in the industry today.
		[Walking an Application](https://tryhackme.com/room/walkinganapplication) - Manually review a web application for security issues using only your browsers developer tools. Hacking with just your browser, no tools or scripts.
		[Content Discovery](https://tryhackme.com/room/contentdiscovery) - Learn the various ways of discovering hidden or private content on a webserver that could lead to new vulnerabilities.
		[Subdomain Enumeration](https://tryhackme.com/room/subdomainenumeration) - Learn the various ways of discovering subdomains to expand your attack surface of a target.
		[Authentication Bypass](https://tryhackme.com/room/authenticationbypass) - Learn how to defeat logins and other authentication mechanisms to allow you access to unpermitted areas.
		[IDOR](https://tryhackme.com/room/idor) - Learn how to find and exploit IDOR vulnerabilities in a web application giving you access to data that you shouldn't have.
		[File Inclusion](https://tryhackme.com/room/fileinc)- This room introduces file inclusion vulnerabilities, including Local File Inclusion (LFI), Remote File Inclusion (RFI), and directory traversal.
		[SSRF](https://tryhackme.com/room/ssrfqi) - Learn how to exploit Server-Side Request Forgery (SSRF) vulnerabilities, allowing you to access internal server resources.
		[Cross-site Scripting](https://tryhackme.com/room/xssgi) - Learn how to detect and exploit XSS vulnerabilities, giving you control of other visitor's browsers.
		[Command Injection] - Learn about a vulnerability allowing you to execute commands through a vulnerable app, and its remediations.
		[SQL injeciton](https://tryhackme.com/room/sqlinjectionlm) - Learn how to detect and exploit SQL Injection vulnerabilities
	[Network Security](https://tryhackme.com/module/network-security) - Learn the basics of passive and active network reconnaissance. Understand how common protocols work and their attack vectors.
		[Passive Reconnaissance](https://tryhackme.com/room/passiverecon) - Learn about the essential tools for passive reconnaissance, such as whois, nslookup, and dig.
		[Active Reconnaissance](https://tryhackme.com/room/activerecon) - Learn how to use simple tools such as traceroute, ping, telnet, and a web browser to gather information.
		[Nmap Live Host Discovery](https://tryhackme.com/room/nmap01) - Learn how to use Nmap to discover live hosts using ARP scan, ICMP scan, and TCP/UDP ping scan.
		[Nmap Basic Port Scans](https://tryhackme.com/room/nmap02) - Learn in-depth how nmap TCP connect scan, TCP SYN port scan, and UDP port scan work.
		[Nmap Advanced Port Scans](https://tryhackme.com/room/nmap03) - Learn advanced techniques such as null, FIN, Xmas, and idle (zombie) scans, spoofing, in addition to FW and IDS evasion.
		[Nmap Post Port Scans](https://tryhackme.com/room/nmap04) - Learn how to leverage Nmap for service and OS detection, use Nmap Scripting Engine (NSE), and save the results.
		[Protocols and Servers](https://tryhackme.com/room/protocolsandservers) - Learn about common protocols such as HTTP, FTP, POP3, SMTP and IMAP, along with related insecurities.
		[Protocals and Servers 2](https://tryhackme.com/room/protocolsandservers2) - Learn about attacks against passwords and cleartext traffic; explore options for mitigation via SSH and SSL/TLS.
		[Net Sec Challenge](https://tryhackme.com/room/netsecchallenge) - Practice the skills you have learned in the Network Security module.
[Red Teaming](https://tryhackme.com/path/outline/redteaming) - The aim of this pathway is to show you how to emulate a potential adversary attack in complex environments. Going beyond penetration testing, you will learn to conduct successful Red Team engagements and challenge the defence capability of your clients.
	Red Team Fundamentals - Learn the core components of a red team engagement, from threat intelligence to OPSEC and C2s.
		[Red Team Fundamentals](https://tryhackme.com/room/redteamfundamentals)	 - Learn about the basics of a red engagement, the main components and stakeholders involved, and how red teaming differs from other cyber security engagements.
		[Red Team Engagements](https://tryhackme.com/room/redteamengagements) - Learn the steps and procedures of a red team engagement, including planning, frameworks, and documentation.
		[Red Team Threat Intel](https://tryhackme.com/room/redteamthreatintel) - Apply threat intelligence to red team engagements and adversary emulation.
		[Red Team OPSEC](https://tryhackme.com/room/opsec) - Learn how to apply Operations Security (OPSEC) process for Red Teams.
		[Intro to C2](https://tryhackme.com/room/introtoc2) - Learn the essentials of Command and Control to help you become a better Red Teamer and simplify your next Red Team assessment!
	Initial Access - Explore the different techniques to gain initial access to a target system and network from a Red Teamer’s perspective.
		[Red Team Recon](https://tryhackme.com/jr/redteamrecon) - Learn how to use DNS, advanced searching, Recon-ng, and Maltego to collect information about your target.
		[Weaponization](https://tryhackme.com/jr/weaponization) - Understand and explore common red teaming weaponization techniques. You will learn to build custom payloads using common methods seen in the industry to get initial access.
		[Password Attacks](https://tryhackme.com/jr/passwordattacks) - This room introduces the fundamental techniques to perform a successful password attack against various services and scenarios.
		[Phishing](https://tryhackme.com/jr/phishingyl) - Learn what phishing is and why it's important to a red team engagement. You will set up phishing infrastructure, write a convincing phishing email and try to trick your target into opening your email in a real-world simulation.
	Post Compromise - Learn about the steps taken by an attacker right after gaining an initial foothold on a network.
		[The Lay of the Land](https://tryhackme.com/room/thelayoftheland) - Learn about and get hands-on with common technologies and security products used in corporate environments; both host and network-based security solutions are covered.
		[Enumeration](https://tryhackme.com/room/enumerationpe) - This room is an introduction to enumeration when approaching an unknown corporate environment.
		[Windows Privilege Escallation](https://tryhackme.com/room/windowsprivesc20) - Learn the fundamentals of Windows privilege escalation techniques.
#### Windows Forensics
[Sysinternals](https://tryhackme.com/room/btsysinternalssg)  - Learn to use the Sysinternals tools to analyze Window systems or applications.
[Core Windows Processes](https://tryhackme.com/room/btwindowsinternals) - Explore the core processes within a Windows operating system and understand what is normal behavior. This foundational knowledge will help you identify malicious processes running on an endpoint!
[Windows Event Logs](https://tryhackme.com/room/windowseventlogs) - Introduction to Windows Event Logs and the tools to query them.

## BlueTeamLabs
The developers of site request that writeups for Challenges and Investigations only be posted after content has been retired and is no longer worth points.
| Challenge | Category | Difficulty | Completed |
| :---: | :---: | :---: | :---: |
| PowerShell Analysis - Keylogger | Reverse Engineering | Easy | 09 MAR 2022 |
| ThePackage | OSINT | Easy | In progress... |

| Investigation | Category | Difficulty | Completed |
| :---: | :---: | :---: | :---: |
| N/A | N/A | N/A | x |

## HackTheBox

## HackThisSite
| Challenges |  |  |  |  |
| :---: | :---: | :---: | :---: | :---: |
| [Basic](https://github.com/dozmert/Security-Labs/blob/main/HackThisSite/Basic/readme.md) | [Reaslistic](https://github.com/dozmert/Security-Labs/blob/main/HackThisSite/Realistic/readme.md) | Application | Programming | Phonephreaking |
| Javascript | Forensic | Extbasic | Stego | IRC |

## Exercises/Miscellaneous
