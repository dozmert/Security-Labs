## TryHackMe - Anthem
> Completed 25th March 2022.

A beginner level Windows machine which proved to be tricky to work through because of my lack of knowledge around web apps. There were a few times where I had to resort to Google for hints and I will make those instances known in the write-up.

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Nessus
- Zaproxy
- cmseek
- Remmina

---
### Methodology
#### Objectives/Flags
1. Flag 1
2. Flag 2
3. Flag 3
4. Flag 3
5. Login credentials
6. user.txt
7. root.txt

---
#### Reconnaissance
First I started of with a straight forward nmap scan. This failed immediately to what was indicated as a possible block of ICMP traffic. It was recommended to add `-Pn` which will set nmap to port-discovery only
```bash
nmap -T4 -Pn <TargetIP>
```
```
Not shown: 973 filtered tcp ports (no-response), 26 closed tcp ports (conn-refused)
PORT     STATE SERVICE
3389/tcp open  ms-wbt-server
```
We can see 3389/TCP open which google reveals is a Windows RDP service.
These results were a little underwhelming, so I decided to check for http services through the web browser.
```
http://<TargetIP>/
```
Immediately I can see that there is in fact a webpage available and the nmap results were not accurate. I had decided to use Nessus in the background to scan for any additional information. Nessus results were far more accurate, showing me the RDP port and the HTTP port open. What was interesting was that the scan had revealed a screenshot from the RDP service which had shown me a Windows login screen with two users `Administrato` and `SG`. 

Moving onto the website I began going through some basic enumeration. Inspecting source code of the webpage had revealed some key pieces of information.
```
http://<TargetIP>/
  flag2
http://<TargetIP>/authors/jane-doe/
  flag3
http://<TargetIP>/robots.txt
  UmbracoIsTheBest!
```
For this box I decided to try and experiment with a web app scanner to help me with the last two flags. I have used Burpsuite in the past however due to only having the Community Edition I opted for an alternative. With high recommendation  I went with OWASP ZAP. This required some setup which I was happy to do as a learning experience
```bash
sudo apt-get install zaproxy
```
Once installed I installed FoxyProxy onto the web browser and set up a custom proxy address to pipe traffic into ZAP.
```
Name: Zaproxy
Address: 127.0.0.1
Port: 8081
```
With that ready to go I launched up ZAP using ```sudo zaproxy``` and enabled the custom-proxy through the browser extension. Once the page was refreshed, I could see ZAP capturing traffic. I used the spidering feature to gather a large list of pages and exported them into a .txt document to reference later. With a larger pool of subpages captures I used the search feature to find more flags using ```TM{```. This revealed two more flags and made the process of searching through the source-code much easier.
```
http://<TargetIP>/archive/we-arehiring
 flag1
http://<TargetIP>/archive/a-cheers-to-our-it-department/
 flag4
```
The Anthem box's objectives asked to complete some additional objectives. One of which is "What CMS is the website using?"-- I didn't know how to spot this as I wasn't aware of the different types of content manager services out there. This ended up being **Umbraco** which I had to google out of suspicion of its reoccurrence in the web-spider. The next objective asked us to find the name of the Administrator and their email address. A blog post on the website by the user Jane Doe listed an email address as `JD@anthem.com`. Using this we could probably assume that the domain's email format is ``<First initial><Last initial>@anthem.com``.  I was legitimately stumped here for some time as I couldn't find any traces of additional names or clues that would lead me to find the Administrator other than a webpage referencing their passing away. I had to look to Google for hints which got me to search up a poem that was written on the page. 
```
Born on a Monday,  
Christened on Tuesday,  
Married on Wednesday,  
Took ill on Thursday,  
Grew worse on Friday,  
Died on Saturday,  
Buried on Sunday.  
That was the end…
```
This revealed the name **Solomon Grundy**. At that point I had made the connection from the screen shot provided by Nessus displaying **SG** and the possible email address. I actually found this to be quite annoying as it was misleading  clue due the context of the blog post and it's author. The administrator's email and password from robots.txt allowed me to login to the CMS portal at ```http://10.10.57.24/umbraco/#/login```.

#### Weaponization 
N/A

#### Delivery
N/A

#### Exploitation
Now that I was done with the early objectives of the box, I as tasked with gaining access to the Anthem machine. After a bit of trial and error on RDP software I settled on using Remmina.
```bash
sudo apt-get install Remmina
  sudo remmina
    <TargetIP>:3389
```
Once prompted for credentials I used the same credentials that were tested on the CMS portal login. This allowed me to successfully make a connection to the box and gain access to a desktop environment.

#### Installation 
N/A

#### Command & Control
N/A

#### Actions on Objective
 Immediately I could see an objective item ```user.txt``` in which I found a flag. 
 ```
 C:/Users/SG/Desktop
    user.txt
 ```
 
 The next step was to attempt to find root.txt which required me to dig through a bid of the C:/ drive. After a while I searching through as many directories as I could I found myself stumped as to where this could've been located. I must admit I turned to google for a hint and immediately face palmed as there was a process I didn't consider taking. 
 ```
 C:/backup
   restore.txt
 ```
  The C:/ contained a folder named ```backup``` which was only visible when revealing hidden files. I will have to consider new search strategies in future when searching for loot. Inside this folder there was a file which the user I was logged onto did not have permissions to read this file. Knowing enough about Windows machines I figured the file-properties was going to be the first place to figure this out.
 ```
 restore.txt
   right-click -> properties
     Security-tab -> Edit
       <MachineHostname> -> Okay -> Apply 
 ```
I must admit I was a little confused as to how easy it was to give myself permission to access this file, I would have considered that this was only controllable by the Administrator account though it may have just been as straight forward as user SG removing their own permission. Once the file was read-able we find a new flag in possible password form.
Using this possible password I dived straight for the Administrator's user files and entered the new password when prompted.
```
C:/Users/Administrator/Desktop
    root.txt
```
Success, the last flag has been found and the box has been completed.

---
### After-action
#### What I've learned
Coming away from this box it felt more as like a puzzle to solve rather than a realism environment. Getting my hands on web-app scanners was a big win for me as I got to explore new recon techniques in web enumeration. ZAP is a tool I will definitely come back to in the future. Lastly, I found that I need to explore new techniques for combing folders in search of file-loot. Simply using a file explorer will get the job done but is slow and could potentially leave me open to missing items. Otherwise, I just need to be mindful in future of hidden-files. 

#### What I would've done differently
Considering I didn't have a lot of web-app experience, my enumeration of the webserver was all over the place as it was just a huge influx of information. It's quite overwhelming but I think in future I will try and approach it more methodically and categorise the information I'm collecting in steps.