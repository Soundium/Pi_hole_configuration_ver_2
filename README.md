               
## How-to: Pi-Hole + DNSCrypt + Anonymized DNS + Cloudflare DoH Setup on Raspberry Pi 4 
          
Follow my instructions and make your setup **trouble-free!**

Want to report any issue? Feel free to file an <a href="https://github.com/Soundium/Pi_hole_configuration_ver_2/issues">issue</a>.

***
**Additional Information**

**dnscrypt-proxy** is a flexible DNS proxy. It runs on your computer or router, and can locally block unwanted content, reveal where your devices are silently sending data to, make applications feel faster by caching DNS responses, and improve security and confidentiality by communicating to upstream DNS servers over secure channels.

**Anonymized DNS** is a lightweight alternative to Tor and SOCKS proxies, dedicated to DNS traffic. They hide the client IP address to DNS resolvers, providing anonymity in addition to confidentiality and integrity.

DNS Anonymization is only compatible with servers supporting the DNSCrypt protocol.

***     
### Donation
All donations are welcome and will help me to maintain this project. Please use "**Sponsor**" button on the top of this page.
***
### My hardware
- <a href="https://www.raspberrypi.org/products/raspberry-pi-4-model-b/">Raspberry Pi 4 Computer Modell B, 4GB RAM</a>
- <a href="https://www.raspberrypi.org/products/poe-hat/">Power over Ethernet (PoE) HAT for Raspberry Pi 4 & 3B+, Rev. 1.01</a>
- <a href="https://www.amazon.de/SanDisk-SDSQXCG-032G-GN6MA-Extreme-Adapter-Schwarz/dp/B06XYHN68L">SanDisk Extreme Pro microSDHC A1 UHS-I U3 Card + Adapter 32GB</a>
- <a href="https://www.jacob.de/produkte/aluminium-gehaeuse-fuer-rpi4-aluc-si-artnr-6009241.html">Aluminium Gehäuse für Raspberry Pi 4 Modell B, silber</a>

### Important informaiton
The Raspberry Pi uses an SD card for local storage, and as such, it's essential to be smart about the SD card you use. I suggest at least a 32GB card, and for a few more dollars, you can get a high endurance or extreme pro card. I like the SanDisk Extreme Pro 32GB micro SDHC card for $12. You won't need the full 32GB of storage, but this allows for additional write locations that can further extend the life of the card. 

***
### Raspberry Pi 4 Installation

 1. Download the latest version of <a href="https://www.raspberrypi.org/downloads/raspbian/">Raspbian Buster with desktop and recommended software</a>. Do NOT unzip it.

2. Download and install <a href="https://www.balena.io/etcher/">Etcher.io</a>, which we will use to write the Raspbian Buster image to the SD card. There are both PC and Mac versions.

3. Connect your card reader and insert the microSD card. Warning: contents will be overwritten!

4. Start Etcher, click "Select Image" and find the Raspbian Buster zip file you downloaded.

5. Click "Flash!" and wait for the zip to be written to the memory card and the validation to complete. If an error occurs, make sure the card reader/card is not locked. If it's not locked, possibly the download is corrupted or not complete. Try and re-download the Raspbian Buster zip. 

**Note:** If you are doing this on a Windows computer, you may get a pop-up about needing to format a drive. This is erroneous, dismiss it and click Cancel.

7. There is a small Fat32 partition in which we need to create a zero byte file called ssh. 
  - **On Windows**, open a command prompt, CD to the Fat32 partition and enter the following command (ignore the output error..that is expected). If you don't see a drive letter associated with the Fat32 partition, open Disk Manager and assign it a letter. Enter:

```
.>ssh
```

- **On a Mac computer** CD to the Fat32 partition (e.g. cd /Volumes/boot) and type: **touch ssh**

9. Cleanly unmount the microSD card. Yes, don't pull it out! Insert the microSD card into the Raspberry Pi. 

10. Connect your Raspberry Pi to a suitable power source. Since there's no power switch, it will start to immediately boot. 

- If you have a monitor and keyboard attached when booting the first time, a helpful GUI wizard will appear to walk you through the configuration of items such as locale, keyboard, timezone, new password, software updates, etc. 

- If you are doing the setup 'headless', wait a couple of minutes for the system to boot. Using a network scanning app find your "Raspberry Pi" IP. Or if you get lucky, you can open a terminal and type **ping raspberry.pi** and see if it responds.

11. SSH into the Raspberry Pi as user '**pi**' and open the configuration tool (default password is **raspberry**):

```
ssh pi@RaspberryIP
sudo raspi-config
```
12. At a minimum, consider configuring the following items with the tool. If you ran through the configuration with the desktop GUI using the keyboard and monitor, most of this would have already been done. 

- Change password (menu 1): very important
- Network options (menu 2): Change hostname (optional) 
- Boot options (menu 3): console autologin (optional, bad for security, good for ease of use)
- Localisation options (menu 4): keyboard layout, timezone (important) 
- Interface options (menu 5): Enable ssh

13. Next, we should configure the Raspberry Pi for a static IP address. You can do this two ways. 
- One, create a reservation in your router (**prefered**)

- or we can configure a static IP directly in the RPi. If you don't want to go the router route, enter the following command:
```
sudo nano dhcpcd.conf
```
Uncomment the line under # Example static IP configuration and fill in the proper IPs. You don't need an IPv6 address so that that line can be left commented. 

Example:
```
interface eth0
static ip_address=192.168.1.11/24
static routers=192.168.1.254
static domain_name_servers=192.168.1.2
```
Save the configuration file and exit nano.
```
CTRL + X then Y and Enter
```
14. Now we need to update all of the packages if you haven't done so during the desktop GUI configuration process. Type:
```
sudo apt-get update
sudo apt-get dist-upgrade
```
Wait for the updates to complete. Reboot after the updates, so type **sudo reboot**.

15. To be more secure and get automated updates, we will install:

- unattended-upgrades
```
sudo apt-get install unattended-upgrades
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```
Add the following two lines just after the origin-Debian section, and comment out the Debian lines. 
```
"origin=Raspbian,codename=${distro_codename},label=Raspbian";
"origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";
```
Save the configuration file and exit nano.
```
CTRL + X then Y and Enter
```
- auto-upgrades
```
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```
Delete the existing lines and paste this in:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::Verbose "1";
APT::Periodic::AutocleanInterval "7";
```
Save the configuration file and exit nano.
```
CTRL + X then Y and Enter
```
To enable unattended updates type:
```
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

16. Update Raspberry Pi 4 EEPROM (Firmware)

From time to time new Raspberry pi 4 EEPROM (firmware) may be available. This procedure will install an auto-updater and keep you on the latest firmware version. Run these commands below. If it says update required, then merely reboot your RPI with **sudo reboot** and the update will be installed. 

```
sudo apt update
sudo apt full-upgrade
sudo apt install rpi-eeprom
sudo rpi-eeprom-update
```

17. Package Removal

Libreoffice is space hog on the Raspberry Pi 4, and likely you won't need it. Plus this makes your backups larger, so let's get rid of it. If you need it, please skip this section.

```
sudo apt-get remove --purge libreoffice*
sudo apt-get clean
sudo apt-get autoremove
```

18. **Installing and Configuring Pi-Hole**

SSH into your RPi and type:
```
curl -sSL https://install.pi-hole.net | bash
```
Walkthrough the text-based wizard and accept all of the default values. When it asks you for which DNS server to use, select one that you feel most comfortable with. Later, we will install and configure Cloudflared DoH, so it doesn't matter what you select now. Make sure at the end you write down the admin console password at the very end of the installer wizard.

Before connecting to the UI, we’ll run a configuration command to set the password to what we want.
```
sudo pihole -a -p
```
This will ask you to set a new password.

19. There are a lot of blocklists out there, but here are a few that should get you around 2M blocked domains. Login to Pi-Hole (http://YourIP/admin), click on Settings, then blocklists. Paste all at once the list below and click on 'Save and Update'. 

```
https://blocklist.site/app/dl/malware
https://blocklist.site/app/dl/ransomware
https://blocklist.site/app/dl/tracking
https://blocklist.site/app/dl/fraud
https://blocklist.site/app/dl/phishing
https://v.firebog.net/hosts/AdguardDNS.txt
https://hosts-file.net/grm.txt
https://reddestdream.github.io/Projects/MinimalHosts/etc/MinimalHostsBlocker/minimalhosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/KADhosts/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/add.Spam/hosts
https://v.firebog.net/hosts/static/w3kbl.txt
https://v.firebog.net/hosts/BillStearns.txt
https://adaway.org/hosts.txt
https://raw.githubusercontent.com/anudeepND/blacklist/master/adservers.txt
https://v.firebog.net/hosts/Easyprivacy.txt
https://raw.githubusercontent.com/quidsup/notrack/master/trackers.txt
https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/spy.txt
https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/SmartTV.txt
https://s3.amazonaws.com/lists.disconnect.me/simple_malvertising.txt
https://www.malwaredomainlist.com/hostslist/hosts.txt
https://dbl.oisd.nl/
https://openphish.com/feed.txt
http://sysctl.org/cameleon/hosts
https://s3.amazonaws.com/lists.disconnect.me/simple_ad.txt
https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt
https://hosts-file.net/ad_servers.txt
https://www.squidblacklist.org/downloads/dg-ads.acl
https://pgl.yoyo.org/adservers/serverlist.php?hostformat=nohtml
https://ransomwaretracker.abuse.ch/downloads/RW_DOMBL.txt
https://ransomwaretracker.abuse.ch/downloads/RW_URLBL.txt
http://theantisocialengineer.com/AntiSocial_Blacklist_Community_V1.txt
https://osint.bambenekconsulting.com/feeds/c2-dommasterlist-high.txt
https://s3.amazonaws.com/lists.disconnect.me/simple_malware.txt
https://isc.sans.edu/feeds/suspiciousdomains_High.txt
https://mirror1.malwaredomains.com/files/justdomains
https://mirror1.malwaredomains.com/files/immortal_domains.txt
http://winhelp2002.mvps.org/hosts.txt
https://www.stopforumspam.com/downloads/toxic_domains_whole.txt
https://raw.githubusercontent.com/Dawsey21/Lists/master/main-blacklist.txt
https://someonewhocares.org/hosts/hosts
https://heuristicsecurity.com/dohservers.txt
https://phishing.army/download/phishing_army_blocklist_extended.txt
https://raw.githubusercontent.com/oneoffdallas/dohservers/master/list.txt
https://raw.githubusercontent.com/CHEF-KOCH/Audio-fingerprint-pages/master/AudioFp.txt
https://raw.githubusercontent.com/CHEF-KOCH/Canvas-fingerprinting-pages/master/Canvas.txt
https://raw.githubusercontent.com/CHEF-KOCH/WebRTC-tracking/master/WebRTC.txt

```

19. With all those blocked domains, there are a few we want whitelisted to prevent possible web surfing issues. So let's install whitelist script. Or don't whitelist anything until you run into a problem and then try to resolve it on the spot.

**### Whitelist script Installation**
- Download
```
cd /opt/
sudo git clone https://github.com/Soundium/Pi_hole_Whitelist.git
```
- Make the script to run the script at 1 AM every day.

`sudo nano /etc/crontab`

- Add this line at the end of the file:       
`0 1 * * *   root    /opt/Pi_hole_Whitelist/scripts/whitelist.sh`

CTRL + X then Y and Enter

- First run
```
sudo /opt/Pi_hole_Whitelist/scripts/whitelist.sh
```  
20. If you want to automate Youtube advertising block, let's install the following script. If you don't need it, please skip this section.

**### Youtube advertising blocker script Installation**

- Register on <a href="https://www.wolframalpha.com/">Wolfram Alpha</a> and get your APPID. 
- Download scripts.
```
cd /opt/
git clone https://github.com/Soundium/Pi_hole_youtube_blocklist.git
cd Pi_hole_youtube_blocklist/scripts
```
- Add your APPID to temp.sh. 
```
sudo nano /opt/Pi_hole_youtube_blocklist/scripts/temp.sh
```
```
# Wolfram Alfa APPID
APPID="Register on https://www.wolframalpha.com/ and put your APPID here"
```
Save the configuration file and exit nano.
```
CTRL + X then Y and Enter
```
- Give the rights.
```
sudo chmod +x /opt/Pi_hole_youtube_blocklist/scripts/temp.sh
```
- Add scripts to crontab to run at 1 AM and 5 AM every day.

`sudo nano /etc/crontab`

- Add those lines at the end of the file:

`0 1 * * *      root    /opt/Pi_hole_youtube_blocklist/scripts/temp.sh`

`0 5 * * *      root    /opt/Pi_hole_youtube_blocklist/scripts/youtube-ads.sh`
```
CTRL + X then Y and Enter
```
- First run
```
sudo Pi_hole_youtube_blocklist/scripts/temp.sh
sudo Pi_hole_youtube_blocklist/scripts/youtube-ads.sh
```
- Add http://localhost/youtube.txt as blacklist from local to Pi-hole setup.

**NOTE** 
If you used all of the block lists above, be prepared to troubleshoot apps or websites that don't work because of blocked domains. If you run across a non-functional site or app, review the Pi-Hole logs for blocked domains and try whitelisting one at a time and re-testing your site/app to see what fixes the problem.

21. We proceed with DNSCrypt installation. Enter the following commands to do a base DNSCrypt installation. Check the latest release <a href="https://github.com/DNSCrypt/dnscrypt-proxy/releases">here</a> and modify the wget and tar commands as needed to use the latest binary.

```
cd /opt

sudo wget https://github.com/jedisct1/dnscrypt-proxy/releases/download/2.0.42/dnscrypt-proxy-linux_arm-2.0.42.tar.gz

sudo tar -xf dnscrypt-proxy-linux_arm-2.0.42.tar.gz
sudo mv linux-arm dnscrypt-proxy && cd dnscrypt-proxy
sudo cp example-dnscrypt-proxy.toml dnscrypt-proxy.toml
sudo nano dnscrypt-proxy.toml
```
Under Global settings add one or more servers. You can use iOS app DNSCloak to sift through the plethora of servers you can use. For example, you can search for DNS servers that block ads, support doh (DNS over HTTPS), view locations, etc. I like Cloudflare DOH. You can also check out the public DNSCrypt server list here and pick one or more that fits your requirements. 

```
server_names = ['cloudflare']
```
Under List of Local addresses change the port number to something you like, above 1024. I'm using 5350 in this example. Pi-Hole will be using port 53 (standard for DNS), so that's why we must use a custom port number for DNSCrypt.

```
listen_addresses = ['127.0.0.1:5350', '[::1]:5350']
```

Change the following:
```
require_dnssec = true   
require_nofilter = false
cache = false   (we will use the Pi-Hole cache)
```
Scroll down to the bottom of the TOML file. For **server_name** add the same server name you used above. For the 'via' servers, review the relay list <a href="https://github.com/DNSCrypt/dnscrypt-resolvers/blob/master/v2/relays.md">here</a>, and pick a couple that suite your needs. I used servers near my house. You may want to use servers in a different country or have other unique requirements. 

```
routes = [
     { server_name='cloudflare', via=['anon-cs-de2', 'anon-cs-nl'] },
 ]
```
Save the configuration file and exit nano.
```
CTRL + X then Y and Enter
```

22. Now we need to start the service and test it to make sure it's working before we configure Pi-Hole to use it.
```
sudo ./dnscrypt-proxy -service install
sudo ./dnscrypt-proxy
sudo ./dnscrypt-proxy -service start
sudo systemctl status dnscrypt-proxy
```
The **sudo ./dnscrypt-proxy** command will provide detailed startup information and return any errors it encounters.  **sudo systemctl status dnscrypt-proxy** does the same for DNScrypt when it's started as a service. Both should have the same output, as shown below. If the Anonymized setting is properly configured those relay servers will be shown in the DNSCrypt output.

To run a quick test that DNSCrypt can perform name resolution type:

```
./dnscrypt-proxy -resolve www.google.com
```

Next, we need to configure Pi-Hole to use this new functionality.

22. We need to edit some things to put this all to work. We'll start by modifying the following file. 

```
sudo nano /etc/dnsmasq.d/01-pihole.conf
```
In this file, we need to comment out the two existing server values, **#server=1.1.1.1 and #server=1.0.0.1**, replace them with **server=127.0.0.1#5350** and add additional command.

Like so:

```
#server=1.1.1.1
#server=1.0.0.1
server=127.0.0.1#5350
server=::1#5350
server=/use-application-dns.net/
```
Save the configuration file and exit nano.
```
CTRL + X then Y and Enter
```
Next, we need to edit the following file:
```
sudo nano /etc/pihole/setupVars.conf
```
Simply comment out the two DNS entries:
```
#PIHOLE_DNS_1=1.1.1.1
#PIHOLE_DNS_2=1.0.0.1
```
Save the configuration file and exit nano.
```
CTRL + X then Y and Enter
```

Last, restart the DNS service:
```
sudo systemctl restart pihole-FTL.service
```
Our Pi-Hole will now send all DNS requests to cloudflared which runs as our DoH proxy over an encrypted tunnel directly to Cloudflare.

We can test this to check our work. Start with https://www.dnsleaktest.com/ --> it will tell us right away. You may see more than one DNS server listed and that’s okay just as long as Cloudflare is listed under ISP. You were successful!

Next, we can test our work against Cloudflare: https://1.1.1.1/help. 

Whatever the steps involved, it's worthwhile to use Pi-Hole as the authoritative DNS server on the network and watch the statistics roll in. This fantastic tool helps improve performance and improves our privacy and that of our friends, teams, and families.

Happy Adblocking and stay safe :-)
   

***
### License
```
MIT License

Copyright (c) 2020 Soundium

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

