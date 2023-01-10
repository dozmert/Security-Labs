## TryHackMe - Windows Fundamentals 1
> Completed 26th October  2022.

An introductory box I wanted to complete on my way to completing the Windows Local Persistance task in the Red Teaming learning path.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Remmina (RDP access)

---
### Methodology
#### Objectives/Flags
1. Complete the tasks

---
##### Task 1 - Introduction to Windows
1. Read above and start the virtual machine.
	N/A

##### Task 2 - Windows Editions
1. What encryption can you enable on Pro that you can't enable in Home?
	BitLocker

##### Task 3 - The Desktop (GUI)
1. Which selection will hide/disable the Search box?
	Hidden
2. Which selection will hide/disable the Task View button?
	Show Task View button
1. Besides Clock and Network, what other icon is visible in the Notification Area?
	Action Center

##### Task 4 - The File System
1. What is the meaning of NTFS?
	New Technology File System

##### Task 5 - The Windows \ System32 Folders
1. What is the system variable for the Windows folder?
	%windir%

##### Task 6 - User Accounts, Profiles, and Permissions
1. What is the name of the other user account?
	tryhackmebilly
2. What groups is this user a member of?
	Remote Desktop Users, Users
1. What built-in account is for guest access to the computer?
	Guest
1. What is the account status?
	Account is disabled

##### Task 7 - User Account Control
1. What does UAC mean?
	User Account Control

##### Task 8 - Settings and the Control Panel
1. In the Control Panel, change the view to **Small icons**. What is the last setting in the Control Panel view?
	Windows Defender Firewall

##### Task 9 - Task Manager
1. What is the keyboard shortcut to open Task Manager?
	Ctrl+Shift+Esc

---
### After-action
#### What I've learned
Local user and groups manager (lusmgr.msc) is an effective administration tool to use.

#### What I would've done differently
N/A

