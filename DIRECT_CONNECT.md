# Slippi Direct Connection Guide
By Nikki and Randy F. Smash

Markdown by prospekt

[Google docs version](https://docs.google.com/document/d/1HhcdCIEZC-FtFEiAMZjyuAuVlTY7-9NyWZWoY09yr0c/edit?usp=sharing)

This guide will cover how to directly connect your Wii to your PC, that is to have the Wii connected to the PC via an Ethernet cable, without a router or Ethernet switch. There are a few things that will be required to successfully achieve this:

*   A modded Wii capable of running homebrew, with Nintendont-Slippi and an unmodified NTSC 1.02 ISO on it
*   A Wiimote and sensor bar
*   A Wii compatible USB to Ethernet adapter, any USB 2.0 AX88772 adapter should work, the Slippi team recommends [this one](https://www.amazon.com/UGREEN-Ethernet-Adapter-Nintendo-Chromebook/dp/B00MYT481C/)
*   An Ethernet cable of appropriate length for your setup
*   A PC with two network adapters, aka two Ethernet ports, or one Ethernet and one WiFi adapter. These can be USB adapters as well
*   Slippi Desktop App installed on the PC

**This guide also assumes that you are using Windows 10 or macOS**, the process should be similar for Windows 7, but we have not tried it.

**This guide will not cover setting up homebrew on your wii.** Please take a look at [Kadano’s guide](https://docs.google.com/document/d/1iaPI7Mb5fCzsLLLuEeQuR9-BeR8AOwvHyU-FM8GKmEs) for that.

If you have additional questions, join our [discord](https://discord.gg/pPfEaW5) and ask in #support-and-bugs

---
**<span style="text-decoration:underline;">Step 1</span>**

Connect your PC to the Internet via one of it’s two network adapters. If you only have one Ethernet adapter then this will need to be done via WiFi.

---
**<span style="text-decoration:underline;">Step 2</span>**

Plug in the Ethernet adapter to the Wii. It does not matter which USB port you use.

Connect the Ethernet adapter on the Wii, to the Ethernet adapter on the PC via the Ethernet cable.

The next set of steps are broken down into Windows and macOS, use the section for your OS

---
**<span style="text-decoration:underline;">Step 3 - Windows</span>**

Open the Network Connections page on your Windows 10 PC. To do this, press the Start button, then open Settings (the gear icon) as shown below:

![alt_text](https://i.ibb.co/WB8BgHX/unnamed.png "image_tooltip")

Then click the Network & Internet option:

![alt_text](https://i.ibb.co/zxHgWRW/unnamed-1.png "image_tooltip")

And from there click the Change Adapter Settings option:

![alt_text](https://i.ibb.co/PFDY1y8/unnamed-2.png "image_tooltip")

---
**<span style="text-decoration:underline;">Step 4 - Windows</span>**

Your Network Connections page should look similar to this:

![alt_text](https://i.ibb.co/tsNw5Bp/unnamed-3.png "image_tooltip")

You may have more network adapters listed, but you should have at least two. At this point you need to right click and open the properties of the adapter that is currently connected to the internet. For me it is the WiFi connection. Once you have the properties open, click the Sharing tab, then check the box that says “Allow other network users to connect through this computer’s Internet connection” as shown:

If you have the Home networking connection drop down box, then you need to set it to the adapter that is connected to your Wii, in order for the PC to share its Internet connection to the Wii. If you do not have this box, it is because you only have one other network adapter and you do not need to worry.



---
**<span style="text-decoration:underline;">Step 3 - macOS</span>**

Navigate to the Sharing section in System Preferences

![alt_text](https://i.ibb.co/yyjHb1y/unnamed-4.png "image_tooltip")

![alt_text](https://i.ibb.co/zPVJWsK/unnamed-5.png "image_tooltip")

---
**<span style="text-decoration:underline;">Step 4 - macOS</span>**

Open the Internet Sharing Service and set the “Share your connection from” setting to the connection that has internet access and then check the connection that is connected to the Wii in the “To computers using” section

![alt_text](https://i.ibb.co/q173msD/unnamed-6.png "image_tooltip")
_My Wii is connected to my Thunderbolt Ethernet adapter and I am connected to the Internet via WiFi_

Finally check the box next to the Internet Sharing Service

![alt_text](https://i.ibb.co/jJL0mnZ/unnamed-7.png "image_tooltip")

---
**<span style="text-decoration:underline;">Step 5</span>**

<b>You do not need to repeat this step after you’ve done it once. Do not clear your settings unless you know you can remake the connection and have a good reason to do so.</b>

Next, you will need to open the Settings menu on the Wii, and under the Internet section, create a Wired connection. Shown below are pictures to illustrate:

![alt_text](https://i.ibb.co/3BtVjy4/unnamed-8.gif "image_tooltip")

![alt_text](https://i.ibb.co/N9grSy8/unnamed-9.gif "image_tooltip")

![alt_text](https://i.ibb.co/GR7GV35/unnamed-10.gif "image_tooltip")

![alt_text](https://i.ibb.co/1JHBnRQ/unnamed-11.gif "image_tooltip")
<br/>
Once you click Wired Connection, it should initiate an internet connection test, and if this guide has been followed faithfully, it should succeed. If it does _not_ succeed, then most like you have incorrectly shared the internet connection of your PC, or have an incompatible adapter for the Wii.

<b>You do not need to repeat this step after you’ve done it once. Do not clear your settings unless you know you can remake the connection and have a good reason to do so.</b>

---
**<span style="text-decoration:underline;">Step 6</span>**

Once you have passed this network connection test on the Wii, launch Nintendont-Slippi via The Homebrew Channel or similar homebrew loading method. Nintendont-Slippi will ask which device (SD/USB) to load from. Select the device that your NTSC 1.02 ISO is on, and navigate to your ISO. Before loading your ISO, press B to open the settings menu. It should look something like this:

![alt_text](https://i.ibb.co/kB1M5dt/unnamed-12.png "image_tooltip")
In this menu, ensure you turn on Slippi Networking, if you do not, you will not be able to mirror to the PC. Slippi File Write will write replays to the USB or SD card on the Wii (whichever one does **NOT **have the ISO on it). This can be enabled with Slippi Networking if you wish. For Controller Fix codes, it is recommended to leave it set to UCF unless you know what you are doing.

---
**<span style="text-decoration:underline;">Step 7</span>**

Once you have enabled the settings that you desire for your ISO, press B again to exit the settings menu, then press A to load your ISO. Once the game loads, open Slippi Launcher (Slippi Desktop App) on your PC and click “Stream From Console.” If everything has gone correctly, your Wii should appear under New Connections via automatic network discovery.

---
**<span style="text-decoration:underline;">Step 8</span>**

When the Wii appears under New Connections, you will need to add it by clicking the + symbol beside it, as shown below:

![alt_text](https://i.ibb.co/3F97M96/unnamed-13.png "image_tooltip")

When you add the connection, a window like this should appear:

![alt_text](https://i.ibb.co/98gz3jF/unnamed-14.png "image_tooltip")

You will need to set the Target Folder for this connection. The target folder is where the replays from the Wii will be saved, and mirroring will not work if this is not set to a valid folder. Be sure to manually create this folder as Slippi will not create it on its own. Real-Time Mode decreases the latency on the mirror to be within a few frames of the console, and is recommended for Ethernet connections. Once you have these set correctly, click Submit.

---
**<span style="text-decoration:underline;">Step 9</span>**

Now you should see the Wii listed under the Connections section of the Slippi Launcher, press Connect and the Mirror button will become available. At this point you can press the Mirror button to start the console mirror!

![alt_text](https://i.ibb.co/Sm1XNv3/unnamed-15.png "image_tooltip")

---

If you have been able to follow this guide and successfully mirror your console, congratulations! 

Please take a moment to consider the magnitude of the Slippi project, and consider becoming a [Patreon](https://www.patreon.com/fizzi36) so that Fizzi can continue to devote as much time as possible to making Slippi the best it can be.
