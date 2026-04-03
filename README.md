# Network-wide-DNS-sinkhole-and-privacy-implementation

Overview:

After deploying new UniFi hardware and implementing VLANs to simulate an enterprise, I decided I next wanted to buy and install a PoE switch to support my growing home lab and set up a Raspberry Pi Zero 2 W to run Pi-Hole as a network-wide DNS sinkhole to block ads across my home network. Along with blocking ads, using a Raspberry Pi as a private DNS server also helps with privacy, and prevents the ISP from logging browsing habits.

## New Hardware:

For this project I bought multiple pieces of hardware, involving the PoE switch, Raspberry Pi, Micro SD card for the Raspberry Pi OS, etc. The biggest thing I learned about the hardware I used for this project was the difference between an OTG ethernet adapter Vs. a PoE splitter. I’ll later explain the circumstances that I learned this difference, but an OTG (On-The-Go) ethernet adapter converts USB data signals into Ethernet signals and swaps the role of a device from a Peripheral (listens to other devices), to a Host (can power and manage other devices) so it can recognize the adapter as a network device and send traffic through it. The one issue with this is that it doesn’t provide power to the device, but only transmits data. A PoE splitter takes a single ethernet cable carrying both data and power from a PoE switch and splits it into 2 separate connections, one for power and one for data, this is especially useful for devices far from a power outlet. In my case I used an OTG ethernet adapter. The full list of hardware is as follows (Figure 1):
1 USW Ultra Ubiquiti PoE Switch
1 Raspberry Pi Zero 2 W
1 GeeekPi Pi Zero 2 W case
1 Micro SD card
1 OTG ethernet adapter
1 SD card reader
1 Cat 5E ethernet cable
1 IJP 27W power supply adapter

<img width="975" height="731" alt="image" src="https://github.com/user-attachments/assets/cc571cbc-9e3d-4100-b208-848afe61354b" />

Figure 1: Hardware used

## Step 1.

The first thing I did was install the PoE switch. As the Dream Machine router itself has no PoE ports, I took the PoE injector my wired AP was using and used it to connect the switch to the router. I then plugged the wired AP into the first port in the PoE switch. An issue that came up when installing the new equipment was the switch was not visible in the UniFi controller although it had a solid white light which meant it was pending adoption. The AP connected to it was flashing blue meaning it was isolated and appeared as disconnected in the UniFi controller. After doing a bit of troubleshooting such as removing the AP from the network to try and reconnect it, I remembered the OSI model and started looking first at the physical layer. As all the cables themselves were working fine as both the switch and AP were receiving power, I next looked at the PoE injector. I found the issue was that I needed to power cycle the PoE injector and the switch popped up ready for adoption in the UniFi controller. This was because when plugging in the switch it immediately started booting, and if the router wasn’t ready to hand out an IP address to it at that specific moment, or if there was some noise on the ethernet cable during the handshake, the switch might have given up asking for an IP. Power cycling the PoE injector forced the switch to rebroadcast its request for an IP, and this time the router was able to assign one. After adopting the switch I was able to re-adopt the wired AP and everything was connected and powered successfully (Figure 2 and 3).

<img width="975" height="170" alt="image" src="https://github.com/user-attachments/assets/311c4499-c28f-4e7d-9531-eea119cbd1f9" />

Figure 2: Successful adoption of PoE switch

<img width="975" height="731" alt="image" src="https://github.com/user-attachments/assets/1941f828-0c9e-4fcd-8dd7-dfda7c4b50a5" />

Figure 3: Switch adopted and powering AP

## Step 2.

After installing and troubleshooting the switch, I began working with the Raspberry Pi itself, starting with mounting the Pi to the case by screwing each of the four corners into their respective slots (Figure 4).

<img width="975" height="731" alt="image" src="https://github.com/user-attachments/assets/ca7e3578-09ff-4e06-9ce5-fd026ec0480e" />

Figure 4: Raspberry Pi mounted to case

## Step 3.

The next step was to flash the Raspberry Pi’s OS to the micro SD card. I did this by first downloading the Raspberry Pi Imager from their software page (Figure 5).

<img width="975" height="507" alt="image" src="https://github.com/user-attachments/assets/026da4c2-e00c-4be4-a20a-073649d02265" />

Figure 5: Downloading Raspberry Pi Imager

## Step 4.

After downloading the Imager, I next put the micro SD card into the included adapter and put the SD card into the reader. After putting the SD card into the reader, I plugged it into a usb port on my desktop (Figure 6).

<img width="975" height="731" alt="image" src="https://github.com/user-attachments/assets/7e375a76-48aa-430d-a860-0cffcd9a134e" />

Figure 6: SD card reader in USB port

## Step 5.

Next, I opened the imager and selected the model of Raspberry Pi I was using, selected the OS I wanted to use, which was the Raspberry Pi OS (64-bit), and selected the storage device I wanted to put the OS on, which was the micro SD card (Figure 7). After that I entered some customization settings such as creating a hostname, setting my time zone, creating a username and password, esntering my Wi-Fi SSID in case I ever wanted to use that instead of an ethernet connection, enabling SSH so I could access and change settings on the Pi making sure to select password authentication, and finally writing the image (Figure 8).

<img width="975" height="693" alt="image" src="https://github.com/user-attachments/assets/31385c82-560b-4677-88d1-e964c43bcaf5" />
 
Figure 7: Selecting storage device for OS

<img width="975" height="691" alt="image" src="https://github.com/user-attachments/assets/ee8befc6-acc1-4dfd-abdf-daf47d05dfb0" />

Figure 8: Writing OS to micro SD card

## Step 6.

After writing the Raspberry Pi OS to the micro SD card, I next ejected the SD card reader, and installed the micro SD card into the slot on the Pi. I connected the Pi first to power, making sure to plug the cable where power was entering from into the usb-c port labled “PWR IN”, and next the cable from the OTG ethernet adapter into the second usb-c port on the Pi (Figure 9). After connecting the Raspberry Pi, I was greeted with a green light on the board showing that it was receiving power.

<img width="975" height="731" alt="image" src="https://github.com/user-attachments/assets/447a561e-2083-401b-a966-55d78b8d8092" />
 
Figure 9: Connected Raspberry Pi

## Step 7.

After successfully connecting the Pi to power and the switch, I closed the case and opened command prompt to SSH into the Raspberry Pi. I did this using the command “ssh justinlawitz@pihole.local” where the syntax was username@hostname.local, and entered my username and password I had set in the Imager (Figure 10).

<img width="975" height="513" alt="image" src="https://github.com/user-attachments/assets/d70f529d-0e04-4c0d-bf4a-42c9df8c876d" />
 
Figure 10: SSH into the Pi

## Step 8.

After successfully SSHing into the Pi, the first command I ran was “curl -sSL https://install.pi-hole.net | bash” to install Pi-Hole. Curl is used to fetch content from a webpage. The -sSL flags are used to make the installations smoother as -s hides the progress bar and error messages, -S tells curl to still show an error message if the download fails, and -L tells curl to follow the redirect id the URL has moved from http to https. The URL is the address where the Pi-Hole installation script is hosted. Finally, after the pipe | which takes output from the command on the left and feeds it to the input of the command on the right, bash is used to tell the Raspberry Pi to execute every line of code it just downloaded. After running the command Pi-Hole began to install (Figure 11).

<img width="975" height="512" alt="image" src="https://github.com/user-attachments/assets/4ad46bc8-929c-44b0-b769-be26f8830e1b" />

Figure 11: Pi-Hole installing

## Step 9.

After Pi-Hole had finished installing, the installer interface opened and it next asked me to set a static IP for the raspberry Pi (Figure 12).

<img width="975" height="513" alt="image" src="https://github.com/user-attachments/assets/e08fbb34-e033-48fb-bd7f-3b1059bebf2c" />

Figure 12: Static IP Needed prompt

## Step 10.

Before setting a static IP for the raspberry Pi, I had remembered that I hadn’t set the VLAN that each port on the new PoE switch should use so it was still on the default VLAN instead of the Private one. To change this, I entered the UniFi controller, selected the switch, and entered the port manager (Figure 13). I then went through each port and set the Native VLAN to the Private one I created in my last project (Figure 14).

<img width="619" height="525" alt="image" src="https://github.com/user-attachments/assets/f4d3f860-24a7-4f48-b5b9-87c0ad20a8e4" />

Figure 13: Switch port manager

<img width="975" height="198" alt="image" src="https://github.com/user-attachments/assets/9451dc8d-ce98-4e72-99ca-94e80db6e8d5" />

Figure 14: Native VLANs changed to Private VLAN

## Step 11.

After configuring the switch to use the Private VLAN as its Native VLAN, I next entered the device manager on the UniFi controller to set the IP address for the Pi as static, or fixed. I did this in the settings of the device, which also confirmed it was in the Private VLAN as the assigned IP for the device had a third octet of 22, which is the ID for the Private VLAN (Figure 15).

<img width="613" height="933" alt="image" src="https://github.com/user-attachments/assets/f99392cd-690c-46b8-a665-75b46bf6e5d9" />

Figure 15: Setting Raspberry Pi to a static IP

## Step 12.

After setting a static IP to the Pi, I next went through the rest of the settings in the installer interface which involved setting ethernet as the primary connection and choosing an upstream DNS provider. To go a bit more in depth on what an “upstream DNS provider” means, Pi-Hole is more of a filter than a phone book when compared to a normal DNS server. Essentially, Pi-Hole knows what to block but doesn’t actually store the IPs and human readable URLs for the entire internet. Instead when searching a legitimate site, it checks its blocklist, sees the site is safe, and then asks the DNS provider for the actual IP address. The upstream provider then sends the IP back to the Pi-Hole, which passes it back to the device asking for the IP of a specific URL. For my upstream provider I chose Cloudflare (DNSSEC) (Figure 16). The next couple configuration options were selecting blocklists for Pi-Hole to use, which initially I just used the default “StevenBlack’s Unified Hosts List”, if I would like to enable query logging, and what settings I wanted to use for the query logging. After those configuration options the installation was complete and I was given a default password to login to the admin page (Figure 17).

<img width="975" height="513" alt="image" src="https://github.com/user-attachments/assets/2fe492eb-1bd0-4eb9-974a-8153ad11c73d" />

Figure 16: Selecting an Upstream DNS Provider

<img width="975" height="512" alt="image" src="https://github.com/user-attachments/assets/3ffa87eb-9fc3-4dce-b509-57ae77ae1b3e" />

Figure 17: Successful installation of Pi-Hole

## Step 13.

After opening the admin page, I found a command listed to set a new password. As I wanted to change it to something I could remember instead of a random string, I SSHed into the Raspberry Pi again and ran the command “sudo pihole setpassword”, and entered my own password (Figure 18).

<img width="975" height="511" alt="image" src="https://github.com/user-attachments/assets/a92b2e64-3a06-4a70-bc01-8834f9ce864d" />

Figure 18: Setting a custom password

## Step 14.

Next, I went back into the UniFi controller and entered the DHCP DNS configurations in the settings of the Private VLAN. I unselected Auto DNS Server and manually added the static IP of the Raspberry Pi to be used as the DNS Server (Figure 19).

<img width="622" height="231" alt="image" src="https://github.com/user-attachments/assets/5b35a0a0-e4a8-44c9-be87-e6159bcf397c" />

Figure 19: Setting DNS server to static IP

## Step 15.

The first way I tested the functionality of Pi-Hole was by using the command “nslookup google.com 192.168.22.199”. This command asks Pi-Hole to tell me the IP for google.com. The first good sign is that in the output of the command, it listed the server used as pi.hole (Figure 20). 

<img width="975" height="405" alt="image" src="https://github.com/user-attachments/assets/d890aac8-414b-43a3-b296-aa0d979233ff" />

Figure 20: Functionality test 1

## Step 16.

Before doing any further testing I also checked speedtest.net as it’s a site that commonly has ads. Although many of the ads were blocked, some made it through the filters of Pi-Hole, specifically ads from Amazon and Prime Video (Figure 21). After a bit of research I learned a bit more on how DNS sinkholes work and first-party vs. third-party ads. Essentially when Pi-Hole blocks an ad, it only blocks ads being served from a blacklisted domain (like doubleclick.net or adserver.com). This can be seen working successfully where the white boxes are, due to the browser asking the DNS for the address of an ad-server, and the Pi-Hole responding with “0.0.0.0” leading to the browser giving up and leaving a blank space. The unblocked ads are from major companies like amazon for instance, and they usually serve ads from their own primary domains like amazon.com. If Pi-Hole blocked amazon.com, that would not only block their ads but all amazons webpages and services. Since Pi-Hole works at the DNS layer, it only sees the “envelope” or domain name and not the “letter” or content that is being served by the domain (Figure 21).

<img width="975" height="442" alt="image" src="https://github.com/user-attachments/assets/1111b678-1774-4649-b39a-6da0fe20cae7" />

Figure 21: Functionality test 2

## Step 17.

After doing a bit of research on why specific ads weren’t being blocked, I wanted to make the blacklist a bit more aggressive and test against common domains that serve ads like doubleclick.net just to confirm that I could block as many ads as possible. Firstly, I logged back into the admin panel for Pi-Hole and added multiple new groups of common ad serving domains I found after a couple short searches (Figure 22). For the final test I decided to run the command “nslookup doubleclick.net 192.168.22.199” in command prompt just to see what a blocked domain would look like in the CLI. The output of the command showed the server being used as pi.hole, and the returned addresses for doubleclick.net being 0.0.0.0 meaning the domain was successfully blocked (Figure 23).

<img width="975" height="510" alt="image" src="https://github.com/user-attachments/assets/7b60d42d-3169-403c-8b4e-58b099689336" />

Figure 22: Adding groups for more aggressive blocking

<img width="975" height="512" alt="image" src="https://github.com/user-attachments/assets/c369f98c-7321-4df0-91d8-04ba06f594aa" />

Figure 23: doubleclick.net blocked

## Step 18.

After returning to the admin page a little over 24 hours later, I found that 2,798 DNS requests were blocked or sink holed from blacklisted domains, and that was about 15.7% of total requests which were unnecessary or intrusive (Figure 24). There were also traffic spikes from around 19:00 to 22:00, which align with times many people in the family are home and browsing/ streaming content that would normally be vulnerable to ads.

<img width="975" height="508" alt="image" src="https://github.com/user-attachments/assets/aea3fb50-f30a-4051-bb39-083821662e84" />

Figure 24: 24 Hours of ad blocking

## Step 19.

The last thing I wanted to do in this project was create firewall rules that would allow both the Private and Guest VLAN to use Pi-hole to block ads instead of just the Private network. To do this I first went into the Guest VLANs settings and changed the DNS Server’s IP to the static IP of the Pi just like I did on the Private VLAN (Figure 25)

<img width="625" height="358" alt="image" src="https://github.com/user-attachments/assets/7c657fc3-b7a2-4b3e-b0b6-ef245780fb53" />

Figure 25: DNS Server settings for Guest VLAN

## Step 20.

After that, I entered the Firewall rules tab in settings in the UniFi controller and clicked Create Policy (Figure 26).

<img width="975" height="230" alt="image" src="https://github.com/user-attachments/assets/74aa4a41-03e9-4520-be06-241799239db5" />

Figure 26: Firewall rules Create Policy

## Step 21.

I set up the firewall rules to essentially punch a hole in the firewall to allow traffic from DNS or port 53. I had to do this because normally VLANs are isolated from each other to prevent lateral movement, and a device on the Guest network wouldn’t be able to reach the Pi-hole on the Private network without these rules. The position of the rules on the list is also important, for instance if the block all rules were above the rules I added on the list, it would override the allow rule and block the traffic from port 53, so I made sure to add the following rules to the top of the list. As for the rules themselves I added two with identical settings but one for the Guest, and one for the Private network. The first thing I did was change the action from Block to Allow, this would clarify that the rule was to allow a certain type of traffic, not block it. In the Destined Zone section I changed External section to Internal as the traffic would be from the Raspberry Pi, a device in the home network and not out on the internet as External would be. I then selected the IP option to clarify the device it should look for and entered the static IP that Pi-hole is using. The last couple settings I changed in the firewall rule was that the traffic it should look for was specific, and on port 53 or DNS, and the protocol it should allow would be TCP/UDP instead of ALL. These last two settings are important due to the Principle of Least Privilege, meaning that a user ot connection should have the bare minimum access necessary to perform its functions, settings like this are important for security and reducing attack surfaces (Figure 27 and 28).

<img width="446" height="958" alt="image" src="https://github.com/user-attachments/assets/afa60b45-aef5-426b-b87b-2e1faa1baa81" /><img width="433" height="956" alt="image" src="https://github.com/user-attachments/assets/adebe238-be47-4be3-ba4a-52fa0da88b08" />

Figure 27: Firewall rule settings

<img width="975" height="302" alt="image" src="https://github.com/user-attachments/assets/e8016eed-3f62-411a-bb9a-53187ac39cef" />

Figure 28: New list of firewall rules

## Step 22. 

Before moving on to changing the Pi-hole configuration files to work on both the Private and Guest network, I had to deal with some troubleshooting as my laptop I would soon do testing on lost connection to the Private network. It turned out this was due to earlier in the lab when I had set the Native VLANs for the new switch I had set the port the AP was plugged into to use the Private VLAN instead of the Default VLAN. Management traffic to and from the AP is tagged with the Default VLAN’s ID, so when the port was set to the Private VLAN’s ID, it couldn’t reach routers management services like DHCP, so my laptop wasn’t assigned an IP as the request never made it to the router. I fixed this by simply changing the Native VLAN on the port the AP was plugged into back to the Default and my laptop was able to wirelessly connect (Figure 29).

<img width="975" height="193" alt="image" src="https://github.com/user-attachments/assets/12172480-ade0-4c77-9b5e-7c7fd7207ede" />

Figure 29: Default Native VLAN for AP

## Step 23.

The final step before the final test to see if Pi-hole would block ads on both networks was to change some configuration files for Pi-hole itself. After SSHing back into the Pi I ran the command “sudo nano /etc/pihole/pihole.toml” to edit the configuration file. In the text editor I changed the listening mode setting to “ALL”, by default Pi-hole uses a setting called “Local” which would only listen for queries coming from the network or VLAN it was connected to like the Private VLAN. Changing listening mode to “ALL” tells the Pi-hole to listen for traffic coming from different VLANs and essentially tells it let the firewall worry about security (Figure 30). The other setting I changed was interface which was originally set to “eth0”, which I deleted and left blank at just “”. Leaving the interface blank ensures the Pi-hole listens to every logical path instead of just ethernet which allows it to hear traffic arriving from different VLANs (Figure 31).

<img width="975" height="550" alt="image" src="https://github.com/user-attachments/assets/a865d50a-048c-464a-a21f-832db4270bb4" />

Figure 30: listeningmode setting

<img width="975" height="551" alt="image" src="https://github.com/user-attachments/assets/3c165774-2c83-499a-9913-b743818a874d" />

Figure 31: interface setting

## Step 24.

The final test I did was using my laptop connected to the Guest network to run the “nslookup doubleclick.com 192.168.22.199” command I ran earlier to make sure that the Pi-hole would block ads on both networks. The output reading 0.0.0.0 in the addresses field meant it was successfully blocked (Figure 32).

<img width="975" height="561" alt="image" src="https://github.com/user-attachments/assets/66340107-c2b7-4ba1-be9c-34a0b9bfbb5d" />

Figure 32: Pi-hole blocking doubleclick.com
