---
layout: single
title: Capture & crack 4-way handshake
date: 2019-03-10 22:52 -0700
---

Today I learned what a four way handshake is and how to capture then crack it.

First lets cover what a four way handshake is. The four way handshake is how your AP and the supplicant connect to eachother. It cannot pass a plaintext password as this is insecure so instead you have a pre-shared-key (PSK) which is your passkey. When this passkey is sent over it is actually hashed and compared with a hash on the AP's end. If correct you are passed through. There is more complexity in the full description but we will start with that.

The first thing we need to do is capture the handshake. To do that we will launch Kali and make your of the aircrack-ng suite.
You will need a wireless adapter to perform these tasks. 
![airmon](../pictures/airmon/airmon2.png)
We confirm we can see our wireless adapter perfect! Next we will  need to put it into monitor mode. To do this simply type
**airmon-ng start wlan0**
![airmon](../pictures/airmon/airmonstart2.png)
You will get some warning about processes that can cause trouble. To alleviate this just type **airmon-ng check kill**
![airmon](../pictures/airmon/airmonkill2.png)
check with airmon-ng to see if we have successfully put it into monitor mode. You should see mon attached at the end of the interface name now.
![airmon](../pictures/airmon/aircheck2.png)
Perfect! We can see that it is in the correct mode. Now we can use airdump to find other AP's in the area. To start this process type **airdump-ng wlan0mon** you should be greeted with a heap of information. It might take a second.
![airmon](../pictures/airmon/airdump2.png)
From here we will pick out our own BSSID and make note of that and the channel it is on. In this case it is **testbssid channel 11** Great! Now we can start a capture with everything related to that channel to try and capture the handshake. To do this we use the command **airodump-ng -c 11 --bssid "bssid" -w capture wlan0mon**. The -c stands for channel the --bssid is self explanatory the -w is going to output a file called capture and the last parameter is the interface to use.
![airmon](../pictures/airmon/airo2.png)
We are going to let that capture while we force the handshake. You could wait it might just take a while. To force it we use **aireplay-ng -0 2 -a "bssid" wlan0mon** . The -0 specifies we are going to use deauthentication packets and 2 is the amount. -a is the bssid and the last parameter is the interface. What this is going to do is kick everyone off the network and force a handshake to happen when they reconnect. It happens almost instantly.
![airmon](../pictures/airmon/aireplay2.png)
We can confirm we got the handshake back on our capture up at the top it should say WPA handshake:

![airmon](../pictures/airmon/handshake2.png)

You can now close the capture. It should inform you how many handshakes were captured. If you type dir you can confirm you do indeed have several dump files. The one we are interested in is the .cap.
![airmon](../pictures/airmon/capture2.png)

The current format is not acceptable to be cracked in hashcat. To fix this we will use a tool called cap2hccapx. You have two options here. There is an online tool to do this for you https://hashcat.net/cap2hccapx/ or you can do it manually. For this I went with manual. You are going to need to download cap2hccapx.c
Go into  a folder location you would like to download it then type
**wget https://raw.githubusercontent.com/hashcat/hashcat-utils/master/src/cap2hccapx.c**
Use **gcc -o cap2hccapx cap2hccapx.c** to convert it into a binary. Once done you can move it into /bin and rm cap2hccapx.c or just work in this folder.
For this next part you are going to need to have hashcat installed. By default you will have it on Kali but because it's a VM I am going to opt for the windows hashcat. Use the following syntax for cracking the hcappx file **hashcat64 -m 2500 crackme.hccapx Wordlist.lst**
![airmon](../pictures/airmon/hashcat.png)
Depending on how fast your gpu is and how complex the password is will determine the time it takes. In this scenario you are only as strong as your wordlist. If somebody uses a complex password this is not going to work. After about 30 seconds hashcat informs me the password has been cracked!
![airmon](../pictures/airmon/hashcracked.png)
For the sake of my routers security I have blurred out the password but you get the idea :).
Had a lof of fun with this one and plan on doing more wireless attacks in the future!


