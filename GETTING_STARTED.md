# Getting Started

## Introduction
This document is meant to assist developers interested in learning about and contributing to the Slippi codebase. 

<i> What this document is: </i> <br>
* An overview of the various components that make up the Slippi Project. 
* A collection of resources and tools. 

<i> What this document is <b> not </b>: </i> <br>
* An in-depth programming tutorial.

## Overview

### The Project
The Slippi project is comprised of a number of different applications, each with their own purpose. Below is an overview of each of these applications function, and their relevant technologies.

<b> [Ishiiruka](https://github.com/project-slippi/Ishiiruka) </b> - A modified version of the Dolphin emulator. This project is responsible for handling things like: communication with the matchmaking server, writing Slippi replay files, passing external data to the emulated game, and playing replays. 
<br> <i> Languages: </i> C++ 

<b> [Slippi SSBM ASM](https://github.com/project-slippi/slippi-ssbm-asm) </b> - A series of ASM mods that are applied to Melee in order make Slippi work.
<br> <i> Languages: </i> PPC Assembly

<b> [Slippi Desktop App](https://github.com/project-slippi/slippi-desktop-app) </b> - A desktop application that allows users to view statstics about previous matches, and launch replays.
<br> <i> Languages: </i> JavaScript
<br> <i> Frameworks: </i> Electron, React

<b> [slippi-js](https://github.com/project-slippi/slippi-js) </b> - A JavaScript library that is used to parse .slp files, and allows for the calculation of game statistics. 
<br> <i> Languages: </i> Typescript

### The Workflow

The user launches <b>Ishiiruka</b> and selects an .iso of Melee to emulate. Upon lauching the emulation of Melee, Ishiiruka injects the modifications made by <b> Slippi SSBM ASM </b>. As the user interacts with the game, information is exchanged between Ishiiruka and the Slippi SSBM ASM code. As a user begins an online match, Ishiiruka starts a log of in-game state reported by Slippi SSBM ASM. This log is ultimately formatted and written to a .slp file, according to the [Slippi replay file spec](https://github.com/project-slippi/slippi-wiki). These files are then viewable from the <b> Slippi Desktop App </b>. From the desktop app, a user may choose to launch the replay, in which case Ishiiruka is launched in a replay mode. A user may also choose to view details or statistics about a previously played match, in order to do so, the desktop app leverages the <b> slippi-js </b> library.  


## Commonly Asked Questions
> "Where is the rollback code located?"  

Rollback is accomplished by work done between the [Slippi SSBM ASM](https://github.com/project-slippi/slippi-ssbm-asm/search?p=1&q=rollback&unscoped_q=rollback) code and the [Ishiiruka](https://github.com/project-slippi/Ishiiruka/search?q=rollback&unscoped_q=rollback) code.

> "How is data moved between the game (assembly) and the emulator?"

Via [EXI communication](https://github.com/project-slippi/Ishiiruka/blob/slippi/Source/Core/Core/HW/EXI_DeviceSlippi.cpp). An example of such is demonstrated in [this video](https://www.youtube.com/watch?v=NOq49h0tkBI) by Fizzi.

> "Where is the matchmaking code?"

The source code for the matchmaking server is in a private repository and is not currently planned to be open sourced.

## Guides & Resources

### Melee Modding and Assembly
* [Fizzi Teaches Basic Assembly and Melee Modding](https://www.youtube.com/watch?v=NOq49h0tkBI) (video) - Recorded by the creator of Slippi, this video teaches some basic assembly and demonstrates EXI communication.
* [Intro to Wii Game Modding by InternetExplorer](https://www.youtube.com/watch?v=IOyQhK2OCs0&list=PL6GfYYW69Pa2L8ZuT5lGrJoC8wOWvbIQv) (video) - Recorded by Dan Salvato - A playlist of videos related to modding Wii games. An excellent resource for debugging with Dolphin.
* [Assembler Tutorial](http://wiibrew.org/wiki/Assembler_Tutorial) - A wii modding specific guide to PPC, and its instructions. 
* [PowerPC Instruction Set](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_71/assembler/idalangref_ins_set.html) - Indepth documentation about PowerPC instructions and their behavior.
* [PowerPC Instruction Set Reference Card](http://www.tentech.ca/downloads/other/PPC_Quick_Ref_Card-Rev1_Oct12_2010.pdf) - An overview of various PowerPC instructions and their behavior.
* [SSBM Data Sheet](https://docs.google.com/spreadsheets/d/1JX2w-r2fuvWuNgGb6D3Cs4wHQKLFegZe2jhbBuIhCG8/preview#gid=12) - A Google Sheet containing information about Melee's memory addresses.
* [MKWii's Go From Noob to Veteran ASM Coder Guide Repo](https://mkwii.com/showthread.php?tid=1114) - A bunch of useful guides to learning ASM and related concepts, not necessarily PPC or Melee specific.

## Tools

### Melee Modding and Assembly
* [VSCode PPC ASM Extension](https://marketplace.visualstudio.com/items?itemName=OGoodness.powerpc-syntax) - A useful extension for syntax highlighting in VSCode
* [HxD](https://mh-nexus.de/en/hxd/) - A free Hex editor. Hex editors in general are useful for looking through memory dumps. 
* [SpeedCrunch](https://speedcrunch.org/) - A calculator for programmers. It allows for quick conversions and operations between hex, binary, octal, and decimal. 
* [Ghidra](https://ghidra-sre.org/) - A tool originally created by the NSA - it's used for reverse engineering programs. Particularly useful, is its ability to generate C code from assembly. 
