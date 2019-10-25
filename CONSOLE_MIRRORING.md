# Slippi Console Mirroring Guide
By Nikki

Markdown by prospekt

[Google docs version](https://docs.google.com/document/d/1ezavBjqVGbVO8aqSa5EHfq7ZflrTCvezRYjOf51MOWg/edit?usp=sharing)

This guide will cover how to setup your Wii for console mirroring.

There are a few things that will be required to setup console mirroring:

* A Wii running homebrew

* A SD card with an unmodified NTSC 1.02 ISO on it

* USB to Ethernet Adapter with a ASIX AX88772 chipset

* We recommend the [UGreen USB 2.0 Ethernet Adapter](https://www.amazon.com/UGREEN-Ethernet-Adapter-Nintendo-Chromebook/dp/B00MYT481C/)

* (Recommended) A USB formatted to some FAT format (FAT16, FAT32, or exFAT)

* A network to connect both your Computer and Wii to using Ethernet

* A Wiimote and Sensor Bar

**This guide will not go over installing Nintendont-Slippi or setting up console replays**. Please read our [Console Replay](https://github.com/project-slippi/project-slippi/wiki/Console-Replay-Guide) guide to set that up.

**This guide will not cover setting up homebrew on your wii.** Please take a look at [Kadano’s guide](https://docs.google.com/document/d/1iaPI7Mb5fCzsLLLuEeQuR9-BeR8AOwvHyU-FM8GKmEs) for that.

If you can’t connect both your computer and Wii using Ethernet to the same network, follow the [Direct Connect](https://docs.google.com/document/d/1HhcdCIEZC-FtFEiAMZjyuAuVlTY7-9NyWZWoY09yr0c/edit?usp=sharing) guide to get around the issue.

If you have additional questions, join our [discord](https://discord.gg/pPfEaW5) and ask in #support-and-bugs

---
**<span  style="text-decoration:underline;">Step 1 - Necessary Software</span>**

 Download the Slippi Desktop App from [https://slippi.gg/downloads](https://slippi.gg/downloads) and install it (or build it if on Linux)

![alt_text](https://i.ibb.co/WNXjZsY/Screen-Shot-2019-09-11-at-5-35-44-PM.png "image_tooltip")
_The first set of links in this image, pick the one for your OS_

Install the Desktop App and then set your Melee NTSC 1.02 ISO in the settings page.

![alt_text](https://i.ibb.co/qNf4GKV/unnamed.png "image_tooltip")
_If your ISO comes up as Unknown please let us know on our Discord._

---
**<span  style="text-decoration:underline;">Step 2 - Setting up the Wii</span>**

Connect your Ethernet adapter to either USB port on your Wii and then plug an Ethernet cable into the adapter and your network.

Navigate to the Internet Settings in the Wii Settings menu and setup a new Wired connection

![alt_text](https://i.ibb.co/rHp1WX3/unnamed-1.png "image_tooltip")
![alt_text](https://i.ibb.co/7KWnwHy/unnamed-2.png "image_tooltip")
![alt_text](https://i.ibb.co/m9WB8Gg/unnamed-3.png "image_tooltip")
![alt_text](https://i.ibb.co/LSPpnxq/unnamed-4.png "image_tooltip")
_Make sure to setup a wired connection, mirroring on WiFi will not yield good results_

![alt_text](https://i.ibb.co/WWQ6PcL/unnamed-5.png "image_tooltip")
_If you see a message similar to this then your connection is ready_

Restart your Wii, head to the Homebrew Channel and wait for the Network icon in the bottom right corner to go solid white.

![alt_text](https://i.ibb.co/5KFsBVS/unnamed-6.png "image_tooltip")

Press start and write down the IP address in the top left corner.

![alt_text](https://i.ibb.co/SPt3gXM/unnamed-7.png "image_tooltip")

Load Nintendont-Slippi

![alt_text](https://i.ibb.co/1rtt5v4/unnamed-8.png "image_tooltip")

Turn on Slippi Networking

![alt_text](https://i.ibb.co/S5fZ6yw/unnamed-9.png "image_tooltip")
_We recommend saving replays to a USB drive as well in case of network failure_

Boot Melee and wait on the Character Select Screen

![alt_text](https://i.ibb.co/wZk2SmB/unnamed-10.png "image_tooltip")

---
**<span  style="text-decoration:underline;">Step 3 - Adding the Console Connection</span>**

Open the Console Mirroring page on the Slippi Desktop App and wait for your console to be discovered

![alt_text](https://i.ibb.co/ZHpRLtP/unnamed-11.png "image_tooltip")
_If the console doesn’t get discovered, you can use the Add Connection button to manually create it._

Press the Add button (plus symbol) and then set the Target Folder and turn on Real-Time Mode.

![alt_text](https://i.ibb.co/h9Qbwy4/unnamed-12.png "image_tooltip")
_We will get to the advanced settings later_

Press Submit and then press Connect on the new Console Connection

![alt_text](https://i.ibb.co/8xmnvvw/unnamed-13.png "image_tooltip")

Once you have connected to the Wii, press the Mirror Button to start a Dolphin mirror

![alt_text](https://i.ibb.co/dWRm2LN/unnamed-14.png "image_tooltip")

---
**<span  style="text-decoration:underline;">Step 4 - Mirroring</span>**

You can now start a game in Melee and the Dolphin will show a mirror of the console 🎉

![alt_text](https://i.ibb.co/6XvNGGY/unnamed-15.png "image_tooltip")
_Replays will be created on your computer in the target folder_

If you have been able to follow this guide and successfully mirror your console, congratulations!

Please take a moment to consider the magnitude of the Slippi project, and consider becoming a [Patreon](https://www.patreon.com/fizzi36) so that Fizzi can continue to devote as much time as possible to making Slippi the best it can be.

---
**<span  style="text-decoration:underline;">Advanced Features</span>**


**<span  style="text-decoration:underline;">Auto switcher</span>**

The auto switcher is designed to be a simple way to setup hiding and showing the Dolphin source in OBS if you plan on using console capture to show the CSS, victory screen, and menus.

It only works with OBS Studio, we cannot not provide it for SLOBS, XSplit, or vMix.

**<span  style="text-decoration:underline;">Step 1</span>**

Install the [OBS Websocket Plugin](https://github.com/Palakis/obs-websocket/releases)

**<span  style="text-decoration:underline;">Step 2</span>**

Open OBS and navigate to the Websocket Server Settings under the Tools Menu

![alt_text](https://i.ibb.co/fx9L6vf/unnamed-16.png "image_tooltip")

Enable the Websocket Server and optionally add a password for safety. I also suggest keeping the System Tray Alerts on so you can know that the Autoswitcher is connected.

![alt_text](https://i.ibb.co/HnmyJ1M/unnamed-17.png "image_tooltip")

**<span  style="text-decoration:underline;">Step 3</span>**

Open the Console Connection page in the Slippi Desktop App and edit your Console Connection

Open the advanced tab and put add in the necessary details

![alt_text](https://i.ibb.co/7NYmtmq/unnamed-18.png "image_tooltip")
_Make sure that you put the IP AND Port using a colon between the two_
_The OBS Source Name should match the source that is capturing Dolphin (case-sensitive)_

**<span  style="text-decoration:underline;">Step 4</span>**

Connect to your Wii while OBS is open and you should see a System Tray Alert

Now the Dolphin source will be shown when a game is active and hidden when a game ends

**<span  style="text-decoration:underline;">Relay</span>**

Because each Wii can only be connected to by a single device, we provide a relay to share the Slippi data stream with other programs. This can be used to power stream overlays, in venue projectors, and more. Enabling the relay is as simple as toggling a switch in the Advanced settings of a Console Connection.

---
**<span  style="text-decoration:underline;">F.A.Qs</span>**

Q: Dolphin isn’t showing up in OBS when I use a Game Capture Source

A: Make sure to run OBS as Admin

Q: When I connect to the Wii, it immediately says Writing file X, but the mirror stays on `Waiting For Game`

A: It is best to connect to the Wii and mirror before any games have been played
Also don’t try to mirror on WiFi. It isn’t consistent and can be very annoying
