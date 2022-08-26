## TryHackMe - [OhSINT](https://tryhackme.com/room/ohsint)
> Completed 16th June 2022.

A quick box that tests OSINT skills. 

---
### Environment
#### Operating System
- Latest kali-VM rolling update

#### Tools
- Exiftool
- Web browser

---
### Methodology
#### Objectives/Flags
1. Answer questions 1-7

---
#### Investiation
First I started off by downloaded the provided file `WindowsXP.jpg` and throwing that into exif examining tools. On my `kali` I was going to use exiftool primarily and then through the web browser I used [Jimpl](https://jimpl.com/)  as a secondary checker.
```bash
sudo apt-get install exiftool
exiftool WindowsXP.jpg
```
![](/TryHackMe/OhSINT/images/OhSINT_001.jpg)
Checking the copyright entry we can see `Owoodflint` listed. If we google this name we find a few pages referencing this particular user.
```
https://oliverwoodflint.wordpress.com/
https://twitter.com/owoodflint
  0x00000000000000000000
  @OWoodflint
https://github.com/OWoodfl1nt
  OWoodfl1nt
    people_finder
```
With the information found on these three web pages, I can answer some of the questions. The information for the first two questions could be found on the user's Twitter page.
```
1. What is this user's avatar of?
  Twitter -> Cat
```

```
2. What city is this person in?
  Github -> London
```

To find the SSID of the WAP he's connected to, we pull a twitter post that appears to state the user's BSSID. We throw BSSID this into a resource called [Wigle](https://wigle.net/) which is used to map global networks.

![](/TryHackMe/OhSINT/images/OhSINT_002.jpg)

![](/TryHackMe/OhSINT/images/OhSINT_003.jpg)

By searching for the BSSID we can confirm that the user is most likely located in London

```
3. What's the SSID of the WAP he connected to?
  UnileverWIfi
```

The next two questions we can find through the Github page found earlier

![](/TryHackMe/OhSINT/images/OhSINT_004.jpg)

```
4. What is his personal email address?
  Github -> OWoodflint@gmail.com

5. What site did you find his email address on?
  Github
```

The last two questions can be answered using information found on the user's Wordpress site. Objective 8 ended up being the toughest information to find and I spent a lot of time blindly searching across the webpages and enumerating further. After spending too long on this I found a hint that indicated that it was hidden on the WordPress site's source.

![](/TryHackMe/OhSINT/images/OhSINT_005.jpg)

![](/TryHackMe/OhSINT/images/OhSINT_006.jpg)

```
6. Where has he gone on holiday?
  Wordpress -> New York
  
8. What is this person's password?
  Wordpress -> pennYDr0pper.!
```

---
### After-action
Firstly I learned that you need a lot of patience when it comes to OSINT. I should have been reading every line of the source instead of jumping over it looking for tags. Secondly both `exiftool` and [Wigle](https://wigle.net/) proved to be useful tools to return to in future.

#### What I would've done differently
I would have liked to have discovered the location of objective 8 myself.
