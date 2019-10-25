# Slippi Console Replay Guide
By Nikki

Markdown version by prospekt

[Google docs version](https://docs.google.com/document/d/1qBMbPiMjm_tDFzh8gizTk01O70TpQpqv41lL25d3ynI/edit?usp=sharing)

This guide will cover how to setup your Wii to record Slippi Replays.

There are a few things that will be required to setup console replays:

*   A Wii running homebrew
*   A SD card with an unmodified NTSC 1.02 ISO on it
*   A USB formatted to some FAT format (FAT16, FAT32, or exFAT)

**This guide will not cover setting up homebrew on your wii.** Please take a look at [Kadano’s guide](https://docs.google.com/document/d/1iaPI7Mb5fCzsLLLuEeQuR9-BeR8AOwvHyU-FM8GKmEs) for that.

**This guide will not cover setting up mirroring with Slippi.** Please take a look at our [Mirroring guide](https://github.com/project-slippi/project-slippi/wiki/Console-Mirroring-Guide) for that.

If you have additional questions, join our [discord](https://discord.gg/pPfEaW5) and ask in #support-and-bugs

---
**<span style="text-decoration:underline;">Step 1 - Necessary Downloads</span>**

Download Nintendont-Slippi from [https://slippi.gg/downloads](https://slippi.gg/downloads)

![alt_text](https://i.ibb.co/p6T5cn9/Screen-Shot-2019-09-11-at-5-35-44-PM.png "image_tooltip")

   _The bottom link in this image_

---
**<span style="text-decoration:underline;">Step 2 - Setting up your SD Card</span>**

Extract the zip onto your SD card.

![alt_text](https://i.ibb.co/DM3rPRV/unnamed-1.png "image_tooltip")
_The zip contains an “apps” folder that can be placed in the root of your SD card_

The Melee ISO should be copied to your SD card to the following path
```
    SD:/games/<pick a folder name>/game.iso
```

![alt_text](https://i.ibb.co/2sjvw37/unnamed-2.png "image_tooltip")
_It should look something like this_

---
**<span style="text-decoration:underline;">Step 3 - wiiconf Settings</span>**

Launch Homebrew channel

![alt_text](https://i.ibb.co/xFH535J/unnamed-3.png "image_tooltip")
_You may have more apps, but these are the important ones_

Load slippi-wiiconf

![alt_text](https://i.ibb.co/5GKpR1r/unnamed-4.png "image_tooltip")
Navigate to Slippi Settings (Network Settings don’t work) and set the time and console name (pick something unique so you can differentiate your replays from those made on different consoles)

![alt_text](https://i.ibb.co/x6fyJC8/unnamed-5.png "image_tooltip")
_The nickname is written into the replay files and the time is used as the file name_

Back out to the homebrew channel

---
**<span style="text-decoration:underline;">Step 4 - Nintendont</span>**

Load Nintendont-Slippi

![alt_text](https://i.ibb.co/1rtt5v4/unnamed-8.png "image_tooltip")

Select SD and then press B to access the settings menu

If your region uses a controller fix (UCF and/or Dween) or Frozen Pokemon stadium, make sure to set the settings here in the Nintendont settings so your replays work with the Slippi Desktop App.

![alt_text](https://i.ibb.co/1QqVDQF/unnamed-7.png "image_tooltip")

Turn on Slippi File Write

![alt_text](https://i.ibb.co/Jt6NMm5/unnamed-8.png "image_tooltip")

---
**<span style="text-decoration:underline;">Step 5 - Slippi Replays</span>**

Boot Melee and replays will be written to your USB 🎉

![alt_text](https://i.ibb.co/wZk2SmB/unnamed-10.png "image_tooltip")

---
If you have been able to follow this guide and successfully mirror your console, congratulations! 

Please take a moment to consider the magnitude of the Slippi project, and consider becoming a [Patreon](https://www.patreon.com/fizzi36) so that Fizzi can continue to devote as much time as possible to making Slippi the best it can be.

---
**<span style="text-decoration:underline;">F.A.Qs</span>**

Q: Can I use one USB for my Melee ISO and one USB for replays?

A: No, you have to use both an SD card and a USB

Q: Can I put my melee ISO on the USB and save replays to the SD card?

A: Yes, that will work fine.

Q: What happens if I unplug my USB?

A: You will need to restart your Wii and re-insert the USB to record more replays

Q: How do I use my replays?

A: Install the Slippi Desktop App from [slippi.gg/downloads](https://slippi.gg/downloads). Plug your USB into your computer and then open any of the replays to view them.
