# Intro
This document will outline the details of the Slippi replay file (.slp). This file is in many ways the heart of Project Slippi and hopefully after reading this document you will be excited to contribute to the future of Melee.

Likely the first important thing to discuss about the file is "why?". Here are a few reasons:
1. **Bootstrapping Development** - Many people are interested in working on Melee related projects. Unfortunately right now the barrier to entry is quite high. A prospective developer would likely have to learn assembly and figure out how to get the information they need out of the game. Being able to describe the contents of a Melee game in an easy to understand file can help to kick-start future Melee software infrastructure.
2. **Common Language** - Having one file structure for games that are played on Dolphin or on console makes it possible for a vibrant ecosystem to grow around it. It doesn't matter how the file is created or generated as long as the data exists in the same form any software designed to deal with that file will work.
3. **Archiving** - It's amazing how much data about Melee games we've lost over the years. Data that has not been collected is non-recoverable. There will never be complex game stats calculated from Mango vs Armada at Genesis 1. Taking the game and converting it to file means that that game can always be replayed, regenerated, or reprocessed. As more software is developed, we can rest easy knowing that any game that has been stored can take advantage of the new advancements.
4. **Sharing** - Capturing and uploading small replay files is much easier than doing the same with large files.

# UBJSON
The overall structure of the file conforms to the Draft 12 version of the [UBJSON spec](http://ubjson.org/). The website does not indicate its version. The raw files for Draft 12 can be found [here](https://github.com/ubjson/universal-binary-json/tree/master/spec12).

UBJSON was chosen for a few reasons:
1. It should be very easy to understand as it greatly resembles JSON
2. It allows binary data to be stored without increasing file size much by taking advantage of the optimized container formats
3. Arbitrary metadata of various types can be easily added and quickly parsed

The .slp file has two core elements: raw and metadata. These elements will always show up in the same order in the file with the raw element first and the metadata element second.

# The `raw` Element
The value for this element is an array of bytes that describe discrete events that were sent by the game to be written. These specific events will be broken down later. The data for the raw element is the largest part of the file and is truly what defines what happened during the game.

The element is defined in optimized format, this is done such that the metadata element can be found and parsed much more easily irrespective of what size it is. The optimized format also enables the type definition to be dropped for each array value which greatly conserves space. You can find more information about optimized container formats in the UBJSON spec.

The lead up to the data will look like the following: `[U][3][r][a][w][[][$][U][#][l][X][X][X][X]`

Each value in brackets is one byte. The first 5 bytes describe the length and name of the key - in this case "raw". The remaining bytes describe the form and length of the value. In this case the value is an array ([) of type uint8 ($U) of length XXXX (lXXXX). The l specifies that the length of the array is described by a long (32-bit number).

It is important to note that while the game is in progress and the data is being actively written, the length will be set to 0. This was done to make overwriting the size easier once all the data has been received.

## Events
As mentioned above, the data contained within the raw element describes specific events that describe what is going on in a Melee game. These events were added in via modding the game and this section will describe what those events are and how to parse the enormous byte array. I will refer to all the bytes in this byte array as the *byte stream*.

Every event is defined by a one byte code followed by a payload. The following table lists all of the existing event types.

| Event Type | Event Code | Description | Added |
| --- | :---: | --- | --- |
| [Event Payloads](#event-payloads) | 0x35 | This event will be the very first event in the byte stream. It enumerates all possible events and their respective payload sizes that may be encountered in the byte stream | 0.1.0
| [Game Start](#game-start) | 0x36 | Contains any information relevant to how the game is set up. Also includes the version of the extraction code | 0.1.0
| [Pre-Frame Update](#pre-frame-update) | 0x37 | One event per frame per character (Ice Climbers are 2 characters). Contains information required to **reconstruct a replay**. Information is collected right before controller inputs are used to figure out the character's next action | 0.1.0
| [Post-Frame Update](#post-frame-update) | 0x38 | One event per frame per character (Ice Climbers are 2 characters). Contains information for **making decisions about game states**, such as computing stats. Information is collected at the end of the Collision detection which is the last consideration of the game engine | 0.1.0
| [Game End](#game-end) | 0x39 | Indicates the end of the game | 0.1.0
| [Frame Start](#frame-start) | 0x3A | This event includes the RNG seed and frame number at the start of a frame's processing | 2.2.0
| [Item Update](#item-update) | 0x3B | One event per frame per item with a maximum of 15 updates per frame. This information can be used for stats, training AIs, or visualization engines to handle items. Items include projectiles like lasers or needles | 3.0.0
| [Frame Bookend](#frame-bookend) | 0x3C | An event that can be used to determine that the entire frame's worth of data has been transferred/processed | 3.0.0
| Gecko List | 0x3D | An event that lists gecko codes. As it can be very large, the list is broken up into multiple messages | 3.3.0
| [FOD Platforms](#fod-platforms) | 0x3F | This event records the height of Fountain of Dreams platforms | 3.18.0
| [Whispy Blow Direction](#whispy-blow-direction) | 0x40 | This event records the direction that Whispy is blowing | 3.18.0
| [Stadium Transformations](#stadium-transformations) | 0x41 | This event records the current Pokemon Stadium transformation | 3.18.0
| [Message Splitter](#message-splitter) | 0x10 | A single part of a large message that has been split out. Currently only applies to Gecko List. | 3.3.0

### Data Types
Ranges are specified in this document with inclusive notation, i.e. [0, 255] means that 0 and 255 are both valid values and so are any values in between.

| Name | Description |
| --- | --- |
| uint8 | An integer type composed of 8 bits. Range is [0, 255]
| uint16 | An integer type composed of 16 bits. Range is [0, 65535]
| uint32 | An integer type composed of 32 bits. Range is [0, 4294967295]
| int8 | An integer type composed of 8 bits. Range is [-128, 127]. Assumed to be [two's complement representation](https://en.wikipedia.org/wiki/Two's_complement)
| int16 | An integer type composed of 16 bits. Range is [-32768, 32767]. Assumed to be [two's complement representation](https://en.wikipedia.org/wiki/Two's_complement)
| int32 | An integer type composed of 32 bits. Range is [-2147483648, 2147483647]. Assumed to be [two's complement representation](https://en.wikipedia.org/wiki/Two's_complement)
| [1] | Indicates an array type containing the specified count of elements
| float | A floating point type of 32 bits. Assumed to be [IEEE 754 binary floating point representation](https://en.wikipedia.org/wiki/IEEE_754)
| bool | A special-purpose uint8 type that indicates it should hold the value 0 or 1
| string | A UBJSON string value
| object | A UBJSON object value

### Endianness
As per the UBJSON spec, all numeric types are written in **big-endian** format. And by a happy coincidence, all numeric values in the game data byte stream are also written in big-endian format.

This means that whether you are looking at the length of the byte stream as described in [The `raw` Element](#the-raw-element) or if you are looking at what stage was selected for the game, the byte order is consistent.

### Backwards Compatibility
In order to maintain backward compatibility it is important that any new fields added to a payload are **added to the end**. Because the payload sizes are provided up-front, a new parser reading an old replay file will still be able to parse the file correctly under the condition that fields have not changed position. The new parser must be able to handle the case where "new" fields are not present in the "old" file.

Maintaining forward compatibility for new events should work similarly. An old parser reading a new file will simply ignore new fields and new events.

### Melee IDs
Some of the values in the event payloads are IDs internal to the game. Luckily for us Dan Salvato and other contributors have documented these pretty extensively in a Google sheet.

* [Character/Stage/Item IDs](https://docs.google.com/spreadsheets/d/1JX2w-r2fuvWuNgGb6D3Cs4wHQKLFegZe2jhbBuIhCG8/preview#gid=20)
* [Action State IDs](https://docs.google.com/spreadsheets/d/1JX2w-r2fuvWuNgGb6D3Cs4wHQKLFegZe2jhbBuIhCG8/preview#gid=13)
* [Character-Specific Action State IDs](https://docs.google.com/spreadsheets/d/1Nu3hSc1U6apOhU4JIJaWRC4Lj0S1inN8BFsq3Y8cFjI/preview)
* [Attack IDs](https://docs.google.com/spreadsheets/d/1spibzWaitiA22s7db1AEw1hqQXzPDNFZHYjc4czv2dc/preview)

### Event Payloads
This event should be the very first event in the byte stream. It enumerates all possible events and their respective payload sizes that may be encountered in the byte stream. An event showing up in here does not imply it will be in the stream, only that if it does occur, it will have the specified size.

| Offset | Name | Type | Description |
| --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x35) The command byte for the event payloads event
| 0x1 | Payload Size | uint8 | The size in bytes of the payload for this event, including this byte (i.e. `3n+1`, where `n` is the number of commands to follow)
| 0x2 + 0x3*i* | Other Command Byte | uint8 | A command byte that may be encountered in the byte stream. *i* is dependent on the payload size, the rest of the payload is all command/size pairs
| 0x3 + 0x3*i* | Other Command Payload Size | uint16 | The size in bytes of the payload for the command (Note: some replays load a large-enough gecko code payload that the size for that event overflows. For message splitter events, length is implied by the [splitter fields](#message-splitter))

### Game Start
This is data that will be transferred as the game is starting. It includes all the information required to initialize the game such as the game mode, settings, characters selected, stage selected. The entire block that contains all of this data is written out in the [`Game Info Block`](#game-info-block) but not all of it is understood/documented. This event will occur exactly once in the stream, immediately after the Event Payloads.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x36) The command byte for the game start event | 0.1.0
| 0x1 | Version | uint8[4] | 4 bytes describing the current extraction code version. `major.minor.build.unused` | 0.1.0
| 0x5 | Game Info Block | uint8[312] | Full game info block that Melee reads from to initialize a game. For a breakdown of the bytes, see the table [Game Info Block](#game-info-block) | 0.1.0
| 0x13D | Random Seed | uint32 | The random seed before the game start | 0.1.0
| 0x141 + 0x8*i* | Dashback Fix | uint32 | Controller fix dashback option (0 = off, 1 = UCF, 2 = Dween). *i* is 0-3 depending on the character port. | 1.0.0
| 0x145 + 0x8*i* | Shield Drop Fix | uint32 | Controller fix shield drop option (0 = off, 1 = UCF, 2 = Dween). *i* is 0-3 depending on the character port. | 1.0.0
| 0x161 + 0x10*i* | Nametag | Shift JIS char16[8] | Nametags used by the players. *i* is 0-3 depending on the character port. Nametags are [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS) encoded. The English characters are full width characters and [can be converted](https://github.com/project-slippi/slp-parser-js/blob/master/src/utils/fullwidth.ts) to normal ASCII/half width characters | 1.3.0
| 0x1A1 | PAL | bool | Value is 1 if PAL is enabled, 0 otherwise | 1.5.0
| 0x1A2 | Frozen PS | bool | Value is 1 if frozen Pokémon Stadium is enabled, 0 otherwise | 2.0.0
| 0x1A3 | Minor Scene | u8 | Minor scene number. Mostly useless atm, should always be 0x2 | 3.7.0
| 0x1A4 | Major Scene | u8 | Major scene number. 0x2 when the game is played from VS mode, 0x8 when online game (has rollbacks) | 3.7.0
| 0x1A5 + 0x1F*i* | Display Name | Shift JIS string | Display names used by the players if using Slippi Online. *i* is 0-3 depending on the character port. [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS) encoded, characters can be mixed half width (1 byte) and full width (2 bytes). Max 15 characters + null terminator | 3.9.0
| 0x221 + 0xA*i* | Connect Code | Shift JIS string | Connect codes used by the players in using Slippi Online. *i* is 0-3 depending on the character port. The `#` is full width (`0x8194`). All other characters are half width (1 byte). Max 7 1 byte characters + 2 byte `#` + null terminator | 3.9.0
| 0x249 + 0x1D*i* | Slippi UID | string | Firebase UIDs of players if using Slippi Online. *i* is 0-3 depending on the character port. Max 28 characters + null terminator | 3.11.0
| 0x2BD | Language Option | u8 | 0 = Japanese, 1 = English. Needed for HRC because stage is different between the languages | 3.12.0
| 0x2BE | Session ID | string | An ID consisting of the mode and time the session started (e.g. `mode.unranked-2022-12-20T06:52:39.18-0`). Max 50 characters + null terminator. Assigned by server and unique for every session. If you want a unique ID for an individual game, append the game number and tiebreak number to the end of this ID. | 3.14.0
| 0x2F1 | Game Number | u32 | For the given Session ID, starts at 1 | 3.14.0
| 0x2F5 | Tiebreaker Number | u32 | For the given Game Number, will be 0 if not a tiebreak game | 3.14.0

#### Game Info Block
Offsets are **indexed from the start of the Game Info Block**. To get the offset of from the [Game Start Block](#game-start), add `0x5` to the offset.

| Offset | Name | Type | Description |
| --- | --- | --- | --- |
| 0x0 | Game Bitfield 1 | uint8 | See the table [Game Bitfield 1](#game-bitfield-1)
| 0x1 | Game Bitfield 2 | uint8 | See the table [Game Bitfield 2](#game-bitfield-2)
| 0x2 | Game Bitfield 3 | uint8 | See the table [Game Bitfield 3](#game-bitfield-3)
| 0x3 | Game Bitfield 4 | uint8 | See the table [Game Bitfield 4](#game-bitfield-4)
| 0x6 | Bomb Rain | uint8 | Value is 0 if bomb rain is disabled. Any other value will cause bombs to start dropping after 20 seconds have elapsed.
| 0x8 | Is Teams | bool | Value is non-zero if teams game, 0 otherwise
| 0xB | Item Spawn Behavior | int8 | Indicates how frequently items spawn. -1 = off, 0 = very low, 1 = low, 2 = medium, 3 = high, 4 = very high, 5-8 = even higher
| 0xC | Self Destruct Score Value | int8 | Indicates how an SD should be interpreted for scoring. Can be -2, -1, or 0 if set by the game
| 0xE | Stage | uint16 | [Stage ID](#melee-ids)
| 0x10 | Game Timer | uint32 | The number of seconds for the timer. Will be specified in this field regardless of game mode
| 0x23 | Item Spawn Bitfield 1 | uint8 | See the table [Item Spawn Bitfield 1](#item-spawn-bitfield-1)
| 0x24 | Item Spawn Bitfield 2 | uint8 | See the table [Item Spawn Bitfield 2](#item-spawn-bitfield-2)
| 0x25 | Item Spawn Bitfield 3 | uint8 | See the table [Item Spawn Bitfield 3](#item-spawn-bitfield-3)
| 0x26 | Item Spawn Bitfield 4 | uint8 | See the table [Item Spawn Bitfield 4](#item-spawn-bitfield-4)
| 0x27 | Item Spawn Bitfield 5 | uint8 | See the table [Item Spawn Bitfield 5](#item-spawn-bitfield-5)
| 0x30 | Damage Ratio | float | Indicates the Damage Ratio
| 0x60 + 0x24*i* | External Character ID | uint8 | The player's [character ID](#melee-ids). *i* is 0-3 depending on the character port. Port 1 is *i* = 0, Port 2 is *i* = 1, and so on. There are 6 characters worth of data present in the block, but only 4 are supported by Slippi.
| 0x61 + 0x24*i* | Player Type | uint8 | 0 = human, 1 = CPU, 2 = demo, 3 = empty
| 0x62 + 0x24*i* | Stock Start Count | uint8 | Stocks this player starts with
| 0x63 + 0x24*i* | Costume Index | uint8 | Indicates which costume index the player used. Does not map to an actual color, e.g. blue Young Link and blue Falcon have different values
| 0x67 + 0x24*i* | Team Shade | uint8 | Indicates coloration changes for multiples of the same character on the same team. 0 = normal, 1 = light, 2 = dark
| 0x68 + 0x24*i* | Handicap | uint8 | The handicap set on the player. Will affect offense/defense ratios later in the player record
| 0x69 + 0x24*i* | Team ID | uint8 | Value only relevant if `is teams` is true. 0 = red, 1 = blue, 2 = green
| 0x6C + 0x24*i* | Player Bitfield | uint8 | See the table [Player Bitfield](#player-bitfield)
| 0x6F + 0x24*i* | CPU Level | uint8 | Indicates what the CPU level is. Still specified on human players
| 0x70 + 0x24*i* | Damage Start | uint16 | This is the percent the fighter's first stock will start at
| 0x72 + 0x24*i* | Damage Spawn | uint16 | This is the percent the fighter's stocks will start at. This value will be used for first stock as well if Damage Start is 0
| 0x78 + 0x24*i* | Offense Ratio | float | Indicates a knockback multiplier when this player hits another
| 0x7C + 0x24*i* | Defense Ratio | float | Indicates a knockback multiplier when this player is hit
| 0x80 + 0x24*i* | Model Scale | float | Indicates a multiplier on the size scaling of the character's model

##### Game Bitfield 1
Found in [Game Info Block](#game-info-block).

| Bit Numbers | Bit Mask | Description |
| --- | --- | --- |
| 1-2 | 0x03 | Behavior of the timer. 0 = no timer, 2 = counting down timer, 3 = counting up timer
| 3-5 | 0x1C | Quantity of character places in the UI
| 6-8 | 0xE0 | Game mode. 0 = time, 1 = stock, 2 = coin, 3 = bonus

##### Game Bitfield 2
Found in [Game Info Block](#game-info-block).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Friendly fire is on
| 2 | 0x02 | Break the Targets / Title Screen Demo
| 3 | 0x04 | Classic Mode / Adventure Mode
| 4 | 0x08 | Home Run Contest / Event Match
| 5 | 0x10 | All-Star in-game / waiting area
| 6 | 0x20 | All-Star in-game / waiting area
| 7 | 0x40 | All-Star in-game
| 8 | 0x80 | All-Star in-game

##### Game Bitfield 3
Found in [Game Info Block](#game-info-block).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Unknown
| 2 | 0x02 | Unknown
| 3 | 0x04 | Unknown
| 4 | 0x08 | Unknown
| 5 | 0x10 | Single-button Mode
| 6 | 0x20 | Unknown
| 7 | 0x40 | Unknown
| 8 | 0x80 | Pause Mode

##### Game Bitfield 4
Found in [Game Info Block](#game-info-block).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Timer UI should be shown and still count during pause
| 2 | 0x02 | Standard play HUD should be hidden during pause
| 3 | 0x04 | LRAStart UI should be shown during pause
| 4 | 0x08 | LRAStart and ZRetry UI is hidden during pause
| 5 | 0x10 | ZRetry UI should be shown during pause
| 6 | 0x20 | Unknown
| 7 | 0x40 | Analog Stick UI should be shown during pause
| 8 | 0x80 | Score Display UI should be shown when not paused

##### Item Spawn Bitfield 1
Found in [Game Info Block](#game-info-block).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Metal Box
| 2 | 0x02 | Cloaking Device
| 3 | 0x04 | Pokéball
| 4 | 0x08 | Unknown
| 5 | 0x10 | Unknown
| 6 | 0x20 | Unknown
| 7 | 0x40 | Unknown
| 8 | 0x80 | Unknown

##### Item Spawn Bitfield 2
Found in [Game Info Block](#game-info-block).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Fan
| 2 | 0x02 | Fire Flower
| 3 | 0x04 | Super Mushroom
| 4 | 0x08 | Poison Mushroom
| 5 | 0x10 | Hammer
| 6 | 0x20 | Warp Star
| 7 | 0x40 | Screw Attack
| 8 | 0x80 | Bunny Hood

##### Item Spawn Bitfield 3
Found in [Game Info Block](#game-info-block).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Ray Gun
| 2 | 0x02 | Freezie
| 3 | 0x04 | Food
| 4 | 0x08 | Motion Sensor Bomb
| 5 | 0x10 | Flipper
| 6 | 0x20 | Super Scope
| 7 | 0x40 | Star Rod
| 8 | 0x80 | Lip's Stick

##### Item Spawn Bitfield 4
Found in [Game Info Block](#game-info-block).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Heart Container
| 2 | 0x02 | Maxim Tomato
| 3 | 0x04 | Starman
| 4 | 0x08 | Home Run Bat
| 5 | 0x10 | Beam Sword
| 6 | 0x20 | Parasol
| 7 | 0x40 | Green Shell
| 8 | 0x80 | Red Shell

##### Item Spawn Bitfield 5
Found in [Game Info Block](#game-info-block).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Capsule
| 2 | 0x02 | Box
| 3 | 0x04 | Barrel
| 4 | 0x08 | Egg (Only Yoshi's Story and Yoshi's Island)
| 5 | 0x10 | Party Ball
| 6 | 0x20 | Barrel Cannon
| 7 | 0x40 | Bob-omb
| 8 | 0x80 | Mr. Saturn

##### Player Bitfield
Found in [Game Info Block](#game-info-block).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Stamina mode
| 2 | 0x02 | Silent character
| 3 | 0x04 | Low gravity
| 4 | 0x08 | Invisible
| 5 | 0x10 | Black stock icon
| 6 | 0x20 | Metal
| 7 | 0x40 | Start the game on the warp-in platform
| 8 | 0x80 | Rumble enabled

### Message Splitter
Some event payload messages are split into smaller messages due to size. Messages are continuously read until the `last message` boolean returns true. Currently the only event payload to use this is the `Gecko Codes` event.

While internal messages contained within a message splitter may have a size defined in payload sizes, that size should be ignored and instead the size implied by the message splitter fields should be used.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x10) The command byte for the message splitter event | 3.3.0
| 0x1 | Fixed Size Block | uint8[512] | The contents of the message - may not evenly contain the message | 3.3.0
| 0x201 | Actual Size | uint16 | The actual number of bytes contained in the section | 3.3.0
| 0x203 | Internal Command | uint8 | The command byte for the internal command | 3.3.0
| 0x204 | Last Message | bool | The boolean representing whether this message is the last | 3.3.0

### Frame Start
Frame start is an event added to transfer RNG seed at the very beginning of a frame's processing to prevent desyncs such as the knockback spiral animation desync.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x3A) The command byte for the frame start event | 2.2.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 2.2.0
| 0x5 | Random Seed | uint32 | The random seed at the start of the frame| 2.2.0
| 0x9 | Scene Frame Counter | uint32 | The scene frame counter. Starts at 0 when the game starts. Continues to count frames even if the game is paused. Can be used to determine if a pause happened and how many frames it lasted by looking for jumps in this value from frame to frame | 3.10.0

### Pre-Frame Update
This event will occur exactly once per frame per character (Ice Climbers are 2 characters). Contains information required to **reconstruct a replay**. Information is collected right before controller inputs are used to figure out the character's next action. The post-frame update contains the appropriate character ID to make sense of the action state present here.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x37) The command byte for the pre-frame update event | 0.1.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 0.1.0
| 0x5 | Player Index | uint8 | Between 0 and 3. Port is index + 1 | 0.1.0
| 0x6 | Is Follower | bool | Value is 1 for Nana and 0 otherwise | 0.1.0
| 0x7 | Random Seed | uint32 | The random seed at this point | 0.1.0
| 0xB | Action State ID | uint16 | Indicates the [action state](#melee-ids) the character is in. Very useful for stats | 0.1.0
| 0xD | X Position | float | X position of character | 0.1.0
| 0x11 | Y Position | float | Y position of character | 0.1.0
| 0x15 | Facing Direction | float | -1 = facing left, +1 = facing right, 0 in some rare cases with items | 0.1.0
| 0x19 | Joystick X | float | Processed analog value of X axis of joystick (range: [-1, 1]) | 0.1.0
| 0x1D | Joystick Y | float | Processed analog value of Y axis of joystick (range: [-1, 1]) | 0.1.0
| 0x21 | C-Stick X | float | Processed analog value of X axis of C-stick (range: [-1, 1]) | 0.1.0
| 0x25 | C-Stick Y | float | Processed analog value of Y axis of C-stick (range: [-1, 1]) | 0.1.0
| 0x29 | Trigger | float | Processed analog value of trigger (range: [0, 1]) | 0.1.0
| 0x2D | Processed Buttons | uint32 | See the table [Processed Buttons](#processed-buttons) | 0.1.0
| 0x31 | Physical Buttons | uint16 | See the table [Physical Buttons](#physical-buttons) | 0.1.0
| 0x33 | Physical L Trigger | float | Physical analog value of L trigger (range: [0, 1]). Useful for IPM | 0.1.0
| 0x37 | Physical R Trigger | float | Physical analog value of R trigger (range: [0, 1]). Useful for IPM | 0.1.0
| 0x3B | X analog for UCF | int8 | Raw X axis analog controller input. Used by UCF dashback code | 1.2.0
| 0x3C | Percent | float | Current damage percent | 1.4.0
| 0x40 | Y analog for UCF | int8 | Raw Y axis analog controller input | 3.15.0
| 0x41 | X c-stick for UCF | int8 | Raw X axis c-stick input. Required for 1.0 cardinals | 3.17.0
| 0x42 | Y c-stick for UCF | int8 | Raw Y axis c-stick input. Required for 1.0 cardinals | 3.17.0

#### Processed Buttons
Look at bits set to see processed buttons pressed. Lower uint16 has the same structure as the `Physical Buttons` field, see table [Physical Buttons](#physical-buttons). Found in [Pre-Frame Update](#pre-frame-update).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1-16 | `---` | See table [Physical Buttons](#physical-buttons)
| 17 | 0x00010000 | Joystick up
| 18 | 0x00020000 | Joystick down
| 19 | 0x00040000 | Joystick left
| 20 | 0x00080000 | Joystick right
| 21 | 0x00100000 | C-stick up
| 22 | 0x00200000 | C-stick down
| 23 | 0x00400000 | C-stick left
| 24 | 0x00800000 | C-stick right
| 25 | 0x01000000 | Unused
| 26 | 0x02000000 | Unused
| 27 | 0x04000000 | Unused
| 28 | 0x08000000 | Unused
| 29 | 0x10000000 | Unused
| 30 | 0x20000000 | Unused
| 31 | 0x40000000 | Unused
| 32 | 0x80000000 | Any trigger

#### Physical Buttons
Use bits set to determine physical buttons pressed. Useful for IPM. Found in [Pre-Frame Update](#pre-frame-update).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x0001 | D-pad left
| 2 | 0x0002 | D-pad right
| 3 | 0x0004 | D-pad down
| 4 | 0x0008 | D-pad up
| 5 | 0x0010 | Z
| 6 | 0x0020 | R trigger digital press
| 7 | 0x0040 | L trigger digital press
| 8 | 0x0080 | Unused
| 9 | 0x0100 | A
| 10 | 0x0200 | B
| 11 | 0x0400 | X
| 12 | 0x0800 | Y
| 13 | 0x1000 | Start
| 14 | 0x2000 | Unused
| 15 | 0x4000 | Unused
| 16 | 0x8000 | Unused

### Post-Frame Update
This event will occur exactly once per frame per character (Ice Climbers are 2 characters). Contains information for **making decisions about game states**, such as computing stats. Information is collected at the end of the Collision detection which is the last consideration of the game engine.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x38) The command byte for the post-frame update event | 0.1.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 0.1.0
| 0x5 | Player Index | uint8 | Between 0 and 3. Port is index + 1 | 0.1.0
| 0x6 | Is Follower | bool | Value is 1 for Nana and 0 otherwise | 0.1.0
| 0x7 | Internal Character ID | uint8 | Internal [character ID](#melee-ids). Can only change throughout game for Zelda/Sheik. Before 1.6.0, check frame -123 to determine if Zelda started as Sheik. After 1.6.0, the game start character ID should be consistent with the started character | 0.1.0
| 0x8 | Action State ID | uint16 | Indicates the [action state](#melee-ids) the character is in. Very useful for stats | 0.1.0
| 0xA | X Position | float | X position of character | 0.1.0
| 0xE | Y Position | float | Y position of character | 0.1.0
| 0x12 | Facing Direction | float | -1 = facing left, +1 = facing right, 0 in some rare cases with items | 0.1.0
| 0x16 | Percent | float | Current damage percent | 0.1.0
| 0x1A | Shield Size | float | Current size of shield (range: [0, 60]) | 0.1.0
| 0x1E | Last Hitting Attack ID | uint8 | [ID](#melee-ids) of the last attack that hit a player. Attacks that "hit" reflectors, counters, or absorbers do not modify this field. Link/Young Link's shield behaves as an absorber. If a reflector is used and the reflected projectile hits the enemy, the ID of the projectile is used and not the ID of the reflector. This field corresponds to move staling. Any attack with ID 1 does not participate in staling. This field is set back to 0 on death | 0.1.0
| 0x1F | Current Combo Count | uint8 | The combo count as defined by the game | 0.1.0
| 0x20 | Last Hit By | uint8 | The player that last hit this player | 0.1.0
| 0x21 | Stocks Remaining | uint8 | Number of stocks remaining | 0.1.0
| 0x22 | Action State Frame Counter | float | Number of frames action state has been active. Can have a fractional component for certain actions | 0.2.0
| 0x26 | State Bit Flags 1 | uint8 | See table [State Bit Flags 1](#state-bit-flags-1) | 2.0.0
| 0x27 | State Bit Flags 2 | uint8 | See table [State Bit Flags 2](#state-bit-flags-2) | 2.0.0
| 0x28 | State Bit Flags 3 | uint8 | See table [State Bit Flags 3](#state-bit-flags-3) | 2.0.0
| 0x29 | State Bit Flags 4 | uint8 | See table [State Bit Flags 4](#state-bit-flags-4) | 2.0.0
| 0x2A | State Bit Flags 5 | uint8 | See table [State Bit Flags 5](#state-bit-flags-5) | 2.0.0
| 0x2B | Misc AS (Hitstun remaining) | float | Can be used for different things. While in hitstun, contains hitstun frames remaining | 2.0.0
| 0x2F | Ground/Air State | bool | 0 = grounded, 1 = airborne | 2.0.0
| 0x30 | Last Ground ID | uint16 | ID of the last ground the character stood on | 2.0.0
| 0x32 | Jumps Remaining | uint8 | Number of jumps remaining | 2.0.0
| 0x33 | L-Cancel Status | uint8 | 0 = none, 1 = successful, 2 = unsuccessful | 2.0.0
| 0x34 | Hurtbox Collision State | uint8 | 0 = vulnerable, 1 = invulnerable, 2 = intangible | 2.1.0
| 0x35 | Self-induced Air x Speed  | float | Negative means left, Positive means right | 3.5.0
| 0x39 | Self-induced y Speed  | float | Negative means down, Positive means up | 3.5.0
| 0x3d | Attack-based x Speed  | float | Negative means left, Positive means right | 3.5.0
| 0x41 | Attack-based y Speed  | float | Negative means down, Positive means up | 3.5.0
| 0x45 | Self-induced Ground x Speed  | float | Negative means left, Positive means right | 3.5.0
| 0x49 | Hitlag frames remaining  | float | 0 means "not in hitlag" | 3.8.0
| 0x4D | Animation Index | uint32 | Indicates the animation the character is in. For Wait: 2 = Wait1, 3 = Wait2, 4 = Wait3 | 3.11.0
| 0x51 | Instance Hit By | uint16 | Instance ID of the player/item that last hit this player | 3.16.0
| 0x53 | Instance ID | uint16 | Serially generated, unique ID for each new action state across all fighters. Resets to 0 temporarily on death. | 3.16.0

#### State Bit Flags 1
Found in [Post-Frame Update](#post-frame-update).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Unknown
| 2 | 0x02 | Is absorber active (e.g. G&W bucket)
| 3 | 0x04 | Unknown
| 4 | 0x08 | Active when reflect does not change projectile ownership (mewtwo side b)
| 5 | 0x10 | Is reflect active
| 6 | 0x20 | Unknown
| 7 | 0x40 | Unknown
| 8 | 0x80 | Unknown

#### State Bit Flags 2
Found in [Post-Frame Update](#post-frame-update).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Unknown
| 2 | 0x02 | Unknown
| 3 | 0x04 | Has temporary intangibility or invincibility from subaction
| 4 | 0x08 | Is fastfalling
| 5 | 0x10 | Is defender in hitlag (does not count shield hitlag)
| 6 | 0x20 | Is in hitlag
| 7 | 0x40 | Unknown
| 8 | 0x80 | Unknown

#### State Bit Flags 3
Found in [Post-Frame Update](#post-frame-update).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Unknown
| 2 | 0x02 | Unknown
| 3 | 0x04 | Is holding another character due to grab/command grab
| 4 | 0x08 | Unknown
| 5 | 0x10 | Unknown
| 6 | 0x20 | Unknown
| 7 | 0x40 | Unknown
| 8 | 0x80 | Is shield active

#### State Bit Flags 4
Found in [Post-Frame Update](#post-frame-update).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Unknown
| 2 | 0x02 | Is in hitstun
| 3 | 0x04 | Owners detection hitbox touching shield bubble
| 4 | 0x08 | Unknown
| 5 | 0x10 | Unknown
| 6 | 0x20 | Powershield active
| 7 | 0x40 | Unknown
| 8 | 0x80 | Unknown

#### State Bit Flags 5
Found in [Post-Frame Update](#post-frame-update).

| Bit Number | Bit Value | Description |
| --- | --- | --- |
| 1 | 0x01 | Unknown
| 2 | 0x02 | Is cloaking device
| 3 | 0x04 | Unknown
| 4 | 0x08 | Is follower (e.g. Nana)
| 5 | 0x10 | Is inactive (zelda/shiek when opposite is in play, 0 stock teammate, etc.) Bit should always be 0 in replays.
| 6 | 0x20 | Unknown
| 7 | 0x40 | Is dead
| 8 | 0x80 | Is offscreen

### Item Update
A maximum of 15 items per frame can have their data extracted. This information can be used for stats, training AIs, or visualization engines to handle items. Note that these aren't just for "items" it also includes all projectiles such as Fox lasers, Sheik needles, etc.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x3B) The command byte for the item update event | 3.0.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 3.0.0
| 0x5 | Type ID | uint16 | The [type of item](#melee-ids) this is | 3.0.0
| 0x7 | State | uint8 | The state the item is in. Mostly undocumented, might differ per type | 3.0.0
| 0x8 | Facing Direction | float | -1 = facing left, +1 = facing right, 0 in some cases | 3.0.0
| 0xC | X Velocity | float | X velocity of item | 3.0.0
| 0x10 | Y Velocity | float | Y velocity of item | 3.0.0
| 0x14 | X Position | float | X position of item | 3.0.0
| 0x18 | Y Position | float | Y position of item | 3.0.0
| 0x1C | Damage Taken | uint16 | Amount of damage an item has taken | 3.0.0
| 0x1E | Expiration Timer | float | Number of frames remaining before item expires. Can go into the negatives for certain items such as Link arrows | 3.0.0
| 0x22 | Spawn ID | uint32 | Auto-incremented number whenever an item spawns: 0, 1, 2, 3, etc | 3.0.0
| 0x26 | Misc #1 | uint8 | Samus missile type (0 = Homing, 1 = Super)| 3.2.0
| 0x27 | Misc #2 | uint8 | [Peach turnip face](#turnip-types) | 3.2.0
| 0x28 | Misc #3 | uint8 | Samus/Mewtwo isLaunched boolean for charge shot | 3.2.0
| 0x29 | Misc #4 | uint8 | Samus/Mewtwo current charged power | 3.2.0
| 0x2A | Owner | int8 | 0-3 for the player that owns the item. -1 when not owned | 3.6.0
| 0x2B | Instance ID | uint16 | Inherited instance ID of the owner. 0 when not owned| 3.16.0

### Turnip Faces
Found in [Item Update](#item-update)
| Value | Smile |
| --- | --- |
| 0 | Smile |
| 1 | T Eyes |
| 2 | Line Eyes|
| 3 | Circle Eyes |
| 4 | Upward Curve |
| 5 | Wink |
| 6 | Dot Eyes |
| 7 | Stitch Face |

### Frame Bookend
The frame bookend is a simple event that can be used to determine that the entire frame's worth of data has been transferred/processed. It is always sent at the very end of the frame's transfer.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x3C) The command byte for the frame bookend event | 3.0.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 3.0.0
| 0x5 | Latest Finalized Frame | int32 | For non-rollback, this should always equal the frame number. For rollback, this indicates the index of a frame which is guaranteed not to happen again (finalized). This is very useful when reading a file in real-time to make sure you are processing "finalized" information | 3.7.0

### FOD Platforms
This event only occurs on Fountain of Dreams, and is sent for each change in platform height. If both platforms are moving, there will be two events per frame.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x3F) The command byte for the FOD platform event | 3.18.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 3.18.0
| 0x5 | Platform | uint8 | Which platform has moved. (0 = Right, 1 = Left) | 3.18.0
| 0x6 | Height | float | The platform's new height | 3.18.0

### Whispy Blow Direction
This event only occurs on Dreamland 64, and is sent whenever Whispy changes blow directions.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x40) The command byte for the blow direction event | 3.18.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 3.18.0
| 0x5 | Direction | uint8 | Which direction Whispy is blowing (0 = None, 1 = Left, 2 = Right) | 3.18.0

### Stadium Transformations
This event only occurs on Pokemon Stadium, and is sent whenever the transformation event or transformation type changes.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x41) The command byte for the blow direction event | 3.18.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 3.18.0
| 0x5 | Transformation Event | uint16 | The subevent for each transformation. (2 = Initialize, 3 = On monitor, 4 = Previous transformation receding, 5 = New transformation rising, 6 = Finalize, 0 = Finished) | 3.18.0
| 0x7 | Transformation Type | uint16 | The current or upcoming transformation. (3 = Fire, 4 = Grass, 5 = Normal, 6 = Rock, 9 = Water) | 3.18.0

### Game End
This event indicates the end of the game has occurred. This event will occur exactly once, as the last event in the stream. (Note: there is an unresolved bug where Game End doesn't appear in some replays!)

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x39) The command byte for the game end event | 0.1.0
| 0x1 | Game End Method | uint8 | See table [Game End Method](#game-end-method) | 0.1.0
| 0x2 | LRAS Initiator | int8 | Index of player that LRAS'd. -1 if not applicable | 2.0.0
| 0x3 | Player Placements | int8[4] | 0-indexed placement positions. -1 if player not in game | 3.13.0

#### Game End Method
The behavior of this field depends on the version. Found in [Game End](#game-end).

| Version | Values |
| --- | --- |
| 0.1.0 | 0 = Unresolved, 3 = resolved
| 2.0.0 | 1 = TIME!, 2 = GAME!, 7 = No Contest

# The `metadata` Element
The metadata element contains any miscellaneous data relevant to the game but not directly provided by Melee. Unlike all the other data defined in this doc, which was basically stored as a binary stream, the data in the metadata element is pure UBJSON.

The metadata element can be read individually to save processing time with a bit of effort. For a complete file, the raw element will indicate its size, meaning the entire data block can be skipped in order to extract just the metadata element.

| Key | Type | Description |
| --- | --- | --- |
| startAt | string | Timestamp of when the game started, ISO 8601 format (e.g. `2018-06-22T07:52:59Z`)
| lastFrame | int32 | The frame number of the last frame of the game. Used to show game duration without parsing entire replay
| players | object | Metadata for the individual players. See table [Players Metadata](#players-metadata)
| playedOn | string | Platform the game was played on (values include `dolphin`, `nintendont`, and `network`)
| consoleNick | string | The name of the console the replay was created on.

#### Players Metadata
| Key | Type | Description |
| --- | --- | --- |
| characters | object | Contains the number of frames for which a character was used. This is mostly useful for Zelda/Sheik
| names | object | Contains the Dolphin netplay name or Slippi Online Display Name depending on which form of netplay was used. Will also contain the connect code for Slippi Online if applicable.

Example: `{'characters': {'18': 5209}, 'names': {'netplay': 'Player', 'code': 'ABCD#0'}}` means the player used Marth (internal character ID `18`) for all `5209` frames of the game and had the Display Name `Player` and Connect Code `ABCD#0`. The netplay name from Dolphin and Display Name from Slippi Online use the same key.
