TechDad Travel Kit (Orange Pi Zero 2W)



The idea here is to turn my OrangePi 0 into a family friendly keychain PC specifically designed for long road trips and extending expensive cruise ship internet access to multiple devices. Rather than paying per device, I can use the travel router to extend a single MAC to all my devices. Having a family 5x's all my expenses, so I'm happy to save where I can. Hopefully this can help you justify spending 30 dollars on a SBC :) In all honesty, I had to think of some way to use the 4gigs ram on this thing!



\## Hardware

\* \*\*SBC:\*\* Orange Pi Zero 2W (4GB RAM)

\* \*\*Case:\*\* Flirc Raspberry Pi Zero Case. https://flirc.tv/products/flirc-raspberrypizero?srsltid=AfmBOoolSYEGm5qYu0Ddw\_m5FegmzRBQSKXJ\_toN7PdsyGXNtXYSKcpU\&variant=43085036519656.



I chose this case because of form factor, obviously, but the case itself is a good, sturdy design with the case itself acting as a heatsink. The GPIO slot allows the antenna from the OrangePi a way out of the case, and the thermal pad lines up spot on with the CPU on the OrangePi. The main drawback is the size of the microUSB ports, which are just slightly too tight for the USB C to fit into. A little bit of filing will take care of that. The rest is plug and play.



As for filing the MicroUSB holes, they only need to be widened marginally - particularly at the lower corners, where some millimeters of material need to be removed to allow for the oval shape of the USB C to slide in comfortably. A dremmel would be ideal for this, but I dont have one. I considered a few options, and your patience level may vary. 120 sandpaper on a small screwdriver, the screwdriver itself, some small, finely threaded screws - I finally went with a high speed drill bit filing out the corners. Be sure you clamp the case before you try this - your drill bit may bite into the aluminum and fling it from your grasp. I was able to do this in a few minutes with a test USB C cable.



++CAUTION++ 1. Secure (clamp) your case before attempting to file or drill it. 2. Be careful of your thermal pad if already applied. 3. Fully clean your case before inserting your SBC! Aluminum is conductive, and can short your board.



\* \*\*Network:\*\*

&nbsp;   \* Internal WiFi: AP Mode (Family LAN)

&nbsp;   \* USB 5G WiFi Dongle: Client Mode (Ship/Hotel Wi-Fi)



\* \*\*Storage:\*\* SD Card. I considered splitting into 2 SD cards, but Armbian is able to handle all the functions nicely.



\## Architecture

\* \*\*OS:\*\* Armbian (Debian Bookworm)

\* \*\*Router Software:\*\* RaspAP (Bridged AP + STA)

\* \*\*Services (Docker):\*\*

&nbsp;   \* `itzg/minecraft-bedrock-server` (Local LAN Party)

&nbsp;   \* `filebrowser/filebrowser` (Photo Backup \& Media Share)

