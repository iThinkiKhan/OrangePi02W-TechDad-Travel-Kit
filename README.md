TechDad Travel Kit (Orange Pi Zero 2W)



The idea here is to turn my OrangePi 0 into a family friendly keychain PC specifically designed for long road trips and extending expensive cruise ship internet access to multiple devices. Rather than paying per device, I can use the travel router to extend a single MAC to all my devices. Having a family 5x's all my expenses, so I'm happy to save where I can. Hopefully this can help you justify spending 30 dollars on a SBC :) In all honesty, I had to think of some way to use the 4gigs ram on this thing!



\## Hardware

\* \*\*SBC:\*\* Orange Pi Zero 2W (4GB RAM)

\* \*\*Case:\*\* Flirc Raspberry Pi Zero Case. https://flirc.tv/products/flirc-raspberrypizero?srsltid=AfmBOoolSYEGm5qYu0Ddw\_m5FegmzRBQSKXJ\_toN7PdsyGXNtXYSKcpU\&variant=43085036519656.



I chose this case because of form factor, obviously, but the case is a sturdy design with the aluminum body acting as a heatsink. The GPIO slot allows the antenna from the OrangePi a way out of the case, and the thermal pad lines up spot on with the CPU on the OrangePi. The main drawback is the size of the microUSB ports, which are just slightly too tight for the USB C to fit into. A little bit of filing will take care of that. The rest is plug and play.



As for filing the MicroUSB holes, they only need to be widened marginally - particularly at the lower corners, where some millimeters of material need to be removed to allow for the oval shape of the USB C to slide in comfortably. A dremmel-style tool would be ideal for this, but I don't have one. I considered a few options, and your patience level may vary. 120 sandpaper on a small screwdriver, the screwdriver itself, some small, finely threaded screws - I finally went with a high speed drill bit filing out the corners. Be sure you clamp the case before you try this - your drill bit may bite into the aluminum and fling it from your grasp. I was able to do this in a few minutes with a test USB C cable to check fit.



++CAUTION++ 1. Secure (clamp) your case before attempting to file or drill it. 2. Be careful of your thermal pad if already applied. 3. Fully clean your case before inserting your SBC! Aluminum is conductive, and can short your board.



\* \*\*Network:\*\*

&nbsp;   \* Internal WiFi: AP Mode (Family LAN)

&nbsp;   \* USB WiFi 6 Dongle: Client Mode (Ship/Hotel Wi-Fi). I chose a BrosTrend adapter, their drivers are well supported and I have used this for RaspberryPi projects with good success. 

Link: https://www.brostrend.com/products/ac5l

Driver link that worked for me: sh -c 'wget linux.brostrend.com/install -O /tmp/install && sh /tmp/install'

\* \*\*Storage:\*\* 32G U1 SD Card. I considered splitting into 2 SD cards for docker and router, but Armbian and the Pis RAM is able to handle all the functions nicely.



\## Architecture

\* \*\*OS:\*\* Armbian (Debian Bookworm, CLI only) Minimal install. This lets us finely control what we are running. Perfect. 

\* \*\*ARMBIAN SETUP:\*\*

After installing the drivers through BrosTrend (very easy to follow steps on their page), I found that the interface was given a long name, something like the chip or driver name. I wanted this changed to WLAN1, and to have the pi automatically select this adapter as preferred internet access (Client mode) whenever it detects it. This was done by creating a configuration file in the network folder: nano /etc/systemd/network/25-wlan1.network. Then fill in: 

[Match]
Name=wlan1

[Network]
DHCP=yes

[DHCPv4]
RouteMetric=100
(wlan0 is currently using Metric 600, so setting this lower means the system will prioritize it if detected)

Because we are using DHCP and want to stick to it (so we can connect to other networks), we need a good way of contacting the Pi. Installing and activating the avahi.daemon takes care of that for us on our own network. You can also set your hostname to something less conspicuous than orangepizero2w. Make sure you make it persistent!

I installed docker, and created a minecraft folder. Inside that, I created a simple .yaml for docker compose. Gemini spit one out in no time, and I'm sure your favorite assistant can too. Make sure that it auto starts the server using restart:unless-stopped. Use the .yaml to run the itzg bedrock container below (docker compose up -d). This will generate a /data file. I constrained my view distance to 7, and set some difficulty options. Whitelisting is an option - and probably preferred. Who knows who will be on the highway playing Minecraft! After using some logs to watch the install I am able to connect to the server using the boards IP, successfully login and play the game. This is running completely on the OrangePi! I dont notice any lag - we will see what the kids say. Lowering view distance or increasing reception could solve this issues if they arise. 

Speaking of issues arising, we need to make sure we can always access the device despite not knowing its ip or even what network its on. To that end, I will enable the onboard wifi as an AP rather than a client - this creates a constant LAN in short range of the device. I will also utilize that LAN for client connection - tablets for minecraft on the road, and a repeater for my network on the cruise, thereby enabling one connection to serve multiple devices (via a smidge of MAC..interference)

That is where RaspAP comes in. It will save us from all the hard work of networking. It will create our LAN/hotspot (using wlan0), hand out IPs, and route traffic to the USB WiFi for internet access. 


\* \*\*Router Software:\*\* RaspAP

Cmd to download and run installer: curl -sL https://install.raspap.com | bash

This will walk you through some settings. Do your thing, but for a travel router I want pretty much everything to be yes. I opted to leave AdBlock and VPNs off, for fear of disturbing the ships WiFi connection. 

IF YOU WANT TO MAINTAIN HEADLESS - When it asks to stop systemd-networkd and resolved, opt for no or immediately lose ssh connection and the ability to download more packages if needed. Install continued anyway. DO NOT RESTART. 

Since we did not disable networkd or resolved, we created a race condition where the old networking program and raspAP will fight for wlan0 on reboot. Lets resolve that before we restart.

#Disable them from starting on boot, but do NOT stop them now
sudo systemctl disable systemd-networkd
sudo systemctl disable systemd-resolved

#Ensure the RaspAP services are set to start
sudo systemctl enable hostapd
sudo systemctl enable dnsmasq


Open a web browser tab and hit http://(current IP wlan1) - this should show you a RaspAP admin panel. We are technically safe to reboot - RaspAP will broadcast an AP from wlan0. I prefer to be extra safe though, so before I rebooted I ensured my /etc/wpa_supplicant/wpa_supplicant.conf had the proper setup for internet access through wlan1 (and my udev rule would still work for naming with systemd disabled). Then, check cat /etc/dhcpcd.conf and make sure wlan0 is set as a static server (internal board wifi) and wlan1 (the dongle) is set as a client for board internet access. Mine was not, so I added nohook wpa_supplicant to the bottom of the wlan0 entry - locking it to AP mode. 

\* \*\*Services (Docker):\*\*

&nbsp;   \* `itzg/minecraft-bedrock-server` (LAN Party)

itzg is great, and Bedrock is easy to set up. I installed the Java edition as well - its the same process. Make sure you limit the ram in your config files - I set my 4gig board to 2.5gigs. This leaves me plenty of overhead for routing, OS, and Docker. 

&nbsp;   \* `filebrowser/filebrowser` (Photo Backup \& Media Share)

