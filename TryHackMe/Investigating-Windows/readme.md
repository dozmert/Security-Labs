## TryHackMe - [Investigating Windows](https://tryhackme.com/room/investigatingwindows)
> Completed 26th August  2022.

This box was the first forensic style challenges I've undertaken. While I've used Windows for many years, it was a little bit of a challenge to find new ways at tracking down the trail/artifacts for the persistent threat.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Remmina
- Event Viewer
- Task Scheduler
- Powershell (ISE)
- Command Prompt
---
### Methodology
#### Objectives/Flags
1. Answer questions 1-16

---
#### Investigation
I started off by connecting to the Windows machine via RDP using `Remmina` which I had installed previously on my Kali OS. As I connected into the machine, I noticed immediately a command prompt window displaying an attempt to establish a connection. At various points during this lab, random scripts and executables would pop up leading me to believe this whatever had attacked this machine was still in play.

To answer the first question, it was going to be a straightforward process of checking the machines details under the `System Information` tool. This information could be found under the `System Summary` section.
```
1. What’s the version and year of the windows machine? -> Windows Server 2016
```

Moving onto the second question we're tasked with finding the last user to log on. This was challenging to narrow down in the `Event Viewer` tool and I wasn't too sure if my method was valid mostly due to finding a lot of system log on entries and was unsure as to how to rule anything out. I had used the following sources to answer this question [1](https://www.lepide.com/how-to/audit-who-logged-into-a-computer-and-when.html) [2](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/basic-audit-logon-events). Once I found my current log on session, I worked backwards in the logs.
```
- Event Viewer > Create a custom view
  - Event ID 
    - By Log: Security
    - Include: 4624
```
```
2. Which user logged in last? -> Administrator
```

Next, we want to find when the last time a specific user logged into the system. Rather than spending too long trying to figure out Event Viewer log filtering, a web resource had pointed me towards the `net` command [1](https://www.prajwaldesai.com/find-user-last-logon-time/). I figured this could have been used to verify Question 1.
```
- Command Prompt
  - net user <user>
```
```
- Start menu > Command Prompt
  - net user john
```
```
3. When did John log onto the system last? -> Last logon 3/2/2019 5:48:32 PM
```

When I had first connected to this machine, an executable had started up detailing an attempt to establish a connection to an IP address. I had not paid attention to this text, so I needed to track down what it was.
First, I checked the common startup and the user startup folders by using the `Run` function to bring up common and user startup directories.
```
- Run > shell:common startup
- C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
```
```
- Run > shell:startup
- C:\Users\<User>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```
These directories were empty so I pressed on the check out the startup tab in `Task Manager`, although I didn't find anything of interest here.
```
- Task Manager > Startup tab
```
I tried checking `Task Scheduler` to see if there was something that was timed to run at the system startup or user logon.
```
- Task Scheduler
```
There appeared to be malicious looking tasks which lead me to a directory found in the root of the storage drive.
```
C:\TMP
```
I found several malicious looking files in here. However, after spending some time digging through what I could using `notepad`, I couldn't find an IP address. I turned to google to find some additional places to check which lead me to the Registry [1](https://attack.mitre.org/techniques/T1547/001/)
```
- Registry Editor
  - HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
     -C:\TMP\p.exe -s \\10.34.2.3 'net user' > C:\TMP\o2.txt
```
This executable and IP address would match what is seen when the machine first starts.
```
4. What IP does the system connect to when it first starts? -> 10.34.2.3
```

We now needed to find what users other than Administrator had administrative privileges. I knew instinctively to check the `Control Panel`.
```
- Control Panel > User accounts > Manage accounts
  - Administrator, Jenny
```
Jenny was one of these users however as there had to be one more, there had to be someplace else to check. Google results had suggested to check for administrative groups.
```
- Computer Management > Local Users and Groups
    - Groups > Administrators
      - Administrator, Jenny and Guest
```
I was surprised that the `Guest` user entry had only showed up under this Groups tool. I'd have to come back to the `Computer Management` tool at a later point to rediscover what it was capable of.
```
5. What two accounts had administrative privileges (other than the Administrator user)? -> Jenny and Guest
```
Going back the `Task Scheduler` tool, we could answer the next question by checking the frequency in which the tasks were to run.
```
6. What’s the name of the scheduled task that is malicious. -> Clean file system
```

Like my process in Question 4, I went back to check. 
Investigating the tasks further revealed the answer under the `Action` tab.
```
7. What file was the task trying to run daily? -> nc.ps1
```

We could also see that this PowerShell script would have arguments tailing it's execution (`-l 1348`). I had followed the `nc.ps1` to its directory to discover it was a `netcat` listener. I would know from experiences that this argument had to be the listening port.
```
8. What port did this file listen locally for? -> 1348
```

Using our `net` command method from Question 3, we checked for Jenny's last logon time.
```
9. When did Jenny last logon? -> Never
```

Examining the creation date of the scheduled tasks and the malicious file directory I had found the `Netcat` script in, they all seemed to suggest the same date.
```
10. At what date did the compromise take place? -> 03/02/2019
```

It was a bit difficult to answer the next question specifically because I wasn't sure what I was looking for through a mess of almost identical entries. I had to rely on the hint to answer this question after creating a custom view in the `Event Viewer` tool and orienting myself on the suspected date of compromise.
```
- Event Viewer > Create a custom view
  - Event ID 
    - By Log: Security
    - Include: 4672
```
```
11. At what time did Windows first assign special privileges to a new logon? -> 04:04:49 PM
```

From experience I was able to identify this tool in the malicious directory. This was evident by the output file under the executable's name.
```
12. What tool was used to get Windows passwords? -> Mimikatz
```

I spent longer on this question then I should have, specifically because I had overlooked the answer on the first go. Initially I had spent time searching through various output files using a regex expression to pick up IPv4 addresses.
```
\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b
```
Next I spent a bit of time scouring the `Event Viewer` logs for outbound connections. Even tried to check the ARP table using `Command Prompt`.
```Shell
arp -a
```
In the end I went back to check the very first place I looked, the `hosts` file. I had not noticed that the IP that was used to resolve to `google.com` just didn't seem right. I believe this is referred to as a `DNS poisoning`.
```
C:\Windows\System32\drivers\etc
  - 76.32.97.132 google.com
```
```
13. What was the attacker's external control and command servers IP? -> 76.32.97.132
```

Next, we were tasks with finding the shell that was uploaded to this server’s website. I was a bit puzzled after having scoured the malicious directory. After some googling, Microsoft documentation had pointed out the default directory for the Internet Information Services (ISS) web server [1](https://docs.microsoft.com/en-us/previous-versions/office/developer/sharepoint-2010/ms474356(v=office.14).
```
- C:\inetpub\wwwroot
  - test.jsp, b.jsp ans shell.gif
```
From experience, I would surmise that the test.jsp file was uploaded initially to see how the web server would react. After which, the b.jsp was uploaded to create a reverse shell. The `shell.gif` file was probably a giveaway.
```
14. What was the extension name of the shell uploaded via the server's website? -> .jsp
```

I was happy with my solution to the next question. However, as I had used an automated method, I didn't quite spend the time learning on how this could have been done manually using `Event Viewer`. After browsing through the `Windows Firewall` tool for Inbound and outbound rules, there were too many to sift through for creation dates and there was no column to display it easily. A website I had found had provided PowerShell script by the user `beardedeagle` to list all the rules and when they were created in order [1](https://superuser.com/questions/747184/is-there-anyway-to-see-when-a-windows-firewall-rule-was-created-enabled-using-po).
```powershell
$Events = Get-WinEvent -ErrorAction SilentlyContinue -FilterHashtable @{logname="Microsoft-Windows-Windows Firewall With Advanced Security/Firewall"; id=2004}

ForEach ($Event in $Events) {
    $eventXML = [xml]$Event.ToXml()
    For ($i=0; $i -lt $eventXML.Event.EventData.Data.Count; $i++) {
        Add-Member -InputObject $Event -MemberType NoteProperty -Force `
            -Name  $eventXML.Event.EventData.Data[$i].name `
            -Value $eventXML.Event.EventData.Data[$i].'#text'
    }
}

$Events | Format-Table -Property TimeCreated,RuleName -AutoSize
```
After all the rules were listed in creation date order, I examined the last rule to be created which was `Allow outside connections for development properties`.
```
15. What was the last port the attacker opened? -> 1337
```

Having answered Question13, I knew what site was targeted in the `hosts` file.
```
16. Check for DNS poisoning, what site was targeted? ->  google.com		
```
---
### After-action
#### What I've learned
With these initial forensics box, I learned a great deal about the various artifacts left behind. There is a great deal left to learn and understand especially when it came to `Event Viewer` logs.

#### What I would've done differently
I would be more careful next time to not overlook details, more specifically regarding the `hosts` file for poisoned sites. I should go the extra mile to check these IPs. Lastly, I would have liked to have better understood how to identify various logs in the `Event Viewer`, like what a special privilege is and how the system logons differed from user logons.