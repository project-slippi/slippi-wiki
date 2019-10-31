# Intro
This document will outline the details of the Slippi replay file (.slp). This file is in many ways the heart of Project Slippi and hopefully after reading this document you will be excited to contribute to the future of Melee.

Likely the first important thing to discuss about the file is "why?". Here are a few reasons:
1. **Bootstrapping Development** - Many people are interested in working on Melee related projects. Unfortunately right now the barrier to entry is quite high. A prospective Melee hacker would likely have to learn assembly and figure out how to get the information they need out of the game. Being able to describe the contents of a Melee game in an easy to understand file can help to kick-start future Melee software infrastructure.
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

# The raw element
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
| Event Payloads | 0x35 | This event will be the very first event in the byte stream. It enumerates all possible events and their respective payload sizes that may be encountered in the byte stream | 0.1.0
| Game Start | 0x36 | Contains any information relevant to how the game is set up. Also includes the version of the extraction code | 0.1.0
| Pre-Frame Update | 0x37 | One event per frame per character (Ice Climbers are 2 characters). Contains information required to **reconstruct a replay**. Information is collected right before controller inputs are used to figure out the character's next action | 0.1.0
| Post-Frame Update | 0x38 | One event per frame per character (Ice Climbers are 2 characters). Contains information for **making decisions about game states**, such as computing stats. Information is collected at the end of the Collision detection which is the last consideration of the game engine | 0.1.0
| Game End | 0x39 | Indicates the end of the game | 0.1.0
| Frame Start | 0x3A | This event includes the RNG seed and frame number at the start of a frame's processing | 2.2.0
| Item Update | 0x3B | One event per frame per item with a maximum of 15 updates per frame. This information can be used for stats, training AIs, or visualization engines to handle items. Items include projectiles like lasers or needles | 3.0.0
| Frame Bookend | 0x3C | An event that can be used to determine that the entire frame's worth of data has been transferred/processed | 3.0.0

### Endianness
As per the UBJSON spec, all integer types are written in **big-endian** format. And by a happy coincidence, all integers in the game data byte stream are also written in big-endian format.

This means that whether you are looking at the length of the byte stream as described in the previous section or if you are looking at what stage was selected for the game, the byte order is consistent.

### Backwards Compatibility
In order to maintain backward compatibility it is important that any new fields added to a payload are **added to the end**. Because the payload sizes are provided up-front, a new parser reading an old replay file will still be able to parse the file correctly under the condition that fields have not changed position. The new parser must be able to handle the case where "new" fields are not present in the "old" file.

Maintaining forward compatibility for new events should work similarly. An old parser reading a new file will simply ignore new fields and new events.

### Melee IDs
Some of the values in the event payloads are IDs internal to the game. Luckily for us Dan Salvato and other contributors have documented these pretty extensively in a Google sheet.

* [Character IDs](https://docs.google.com/spreadsheets/d/1JX2w-r2fuvWuNgGb6D3Cs4wHQKLFegZe2jhbBuIhCG8/edit#gid=20)
* [Action State IDs](https://docs.google.com/spreadsheets/d/1JX2w-r2fuvWuNgGb6D3Cs4wHQKLFegZe2jhbBuIhCG8/edit#gid=13)
* [Character-Specific Action State IDs](https://docs.google.com/spreadsheets/d/1Nu3hSc1U6apOhU4JIJaWRC4Lj0S1inN8BFsq3Y8cFjI/edit)

### Event Payloads
This event should be the very first event in the byte stream. It enumerates all possible events and their respective payload sizes that may be encountered in the byte stream. An event showing up in here does not imply it will be in the stream, only that if it does occur, it will have the specified size.

| Offset | Name | Type | Description |
| --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x35) The command byte for the event payloads event |
| 0x1 | Payload Size | uint8 | The size in bytes of the payload for this event, including this byte (i.e. `3n+1`, where `n` is the number of commands to follow) |
| 0x2 + 0x3*i* | Other Command Byte | uint8 | A command byte that may be encountered in the byte stream. *i* is dependent on the payload size, the rest of the payload is all command/size pairs |
| 0x3 + 0x3*i* | Other Command Payload Size | uint16 | The size in bytes of the payload for the command |

### Game Start
This is data that will be transferred as the game is starting. It includes all the information required to initialize the game such as the game mode, settings, characters selected, stage selected. The entire block that contains all of this data is written out in the `Game Info Block` but not all of it is understood/documented. This event will occur exactly once in the stream.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x36) The command byte for the game start event | 0.1.0
| 0x1 | Version | uint8[4] | 4 bytes describing the current extraction code version. `major.minor.build.unused` | 0.1.0
| 0x5 | Game Info Block | uint8[312] | Full game info block that Melee reads from to initialize a game. For a breakdown of the bytes, see the table Game Info Block | 0.1.0
| 0x13D | Random Seed | uint32 | The random seed before the game start | 0.1.0
| 0x141 + 0x8*i* | Dashback Fix | uint32 | Controller fix dashback option (0=off, 1=UCF, 2=Dween). *i* is 0-3 depending on the character port. | 1.0.0
| 0x145 + 0x8*i* | Shield Drop Fix | uint32 | Controller fix shield drop option (0=off, 1=UCF, 2=Dween). *i* is 0-3 depending on the character port. | 1.0.0
| 0x161 + 0x10*i* | Nametag | uint16[8] | Nametags used by the players. *i* is 0-3 depending on the character port. | 1.3.0
| 0x1A1 | PAL | bool | Value is 1 if PAL is enabled, 0 otherwise | 1.5.0
| 0x1A2 | Frozen PS | bool | Value is 1 if frozen stadium is enabled, 0 otherwise | 2.0.0

#### Game Info Block
Offsets are **indexed from the Game Start command byte** described above, they are not indexed from the start of the Game Info Block.

| Offset | Name | Type | Description |
| --- | --- | --- | --- |
| 0x5 | Game Bitfield 1 | uint8 | Bits 1 and 2 describe the behavior of the timer. No timer = 0, counting down timer = 2, counting up timer = 3. Bits 3-5 when shifted twice to the right will describe the quantity of character places in the UI. Bits 6-8 when shifted five times to the right will describe the game mode. 0 = time, 1 = stock, 2 = coin, 3 = bonus
| 0x6 | Game Bitfield 2 | uint8 | Bit 1 indicates that Friendly Fire is on (0x01)
| 0x8 | Game Bitfield 3 | uint8 | Bit 2 indicates that the standard play HUD should be hidden during pause (0x02). Bit 3 indicates that the LRAStart UI should be shown during pause (0x04). Bit 4 indicates that pause is disabled (0x08). Bit 7 indicates that the analog stick should be shown on the pause UI (0x40)
| 0xD | Is Teams | bool | Value is 1 if teams game, 0 otherwise
| 0x10 | Item Spawn Behavior | int8 | Indicates how frequently items spawn. -1 = off, 0 = very low, 1 = low, 2 = medium, 3 = high, 4 = very high
| 0x11 | Self Destruct Score Value | int8 | Indicates how an SD should be interpreted for scoring. Can be -2, -1, or 0 if set by the game
| 0x13 | Stage | uint16 | Stage ID
| 0x15 | Game Timer | uint32 | The number of seconds for the timer. Will be specified in this field regardless of game mode
| 0x28 | Item Spawn Bitfield 1 | uint8 | Bits 4-8 are unknown. Pokéball = 0x04, Cloaking Device = 0x02, Metal Box = 0x01
| 0x29 | Item Spawn Bitfield 2 | uint8 | Bunny Hood = 0x80, Screw Attack = 0x40, Warp Star = 0x20, Hammer = 0x10, Poison Mushroom = 0x08, Super Mushroom = 0x04, Fire Flower = 0x02, Fan = 0x01
| 0x2A | Item Spawn Bitfield 3 | uint8 | Lip's Stick = 0x80, Star Rod = 0x40, Super Scope = 0x20, Flipper = 0x10, Motion Sensor Bomb = 0x08, Food = 0x04, Freezie = 0x02, Ray Gun = 0x01
| 0x2B | Item Spawn Bitfield 4 | uint8 | Red Shell = 0x80, Green Shell = 0x40, Parasol = 0x20, Beam Sword = 0x10, Home Run Bat = 0x08, Starman = 0x04, Maxim Tomato = 0x02, Heart Container = 0x01
| 0x2C | Item Spawn Bitfield 5 | uint8 | Mr. Saturn = 0x80, Bob-omb = 0x40, Barrel Cannon = 0x20, Party Ball = 0x10, Bits 5-8 are unknown
| 0x35 | Damage Ratio | float | Indicates the Damage Ratio
| 0x65 + 0x24*i* | External Character ID | uint8 | The player's character ID. *i* is 0-3 depending on the character port. Port 1 is *i* = 0, Port 2 is *i* = 1, and so on. There are 6 characters worth of data present in the block, but only 4 are supported by Slippi.
| 0x66 + 0x24*i* | Player Type | uint8 | 0 = human, 1 = CPU, 2 = demo, 3 = empty
| 0x67 + 0x24*i* | Stock Start Count | uint8 | Stocks this player starts with
| 0x68 + 0x24*i* | Costume Index | uint8 | Indicates which costume index the player used. Does not map to an actual color, e.g. blue Young Link and blue Falcon have different values
| 0x6C + 0x24*i* | Team Shade | uint8 | Indicates coloration changes for multiples of the same character on the same team. 0 = normal, 1 = light, 2 = dark
| 0x6D + 0x24*i* | Handicap | uint8 | The handicap set on the player. Will affect offense/defense ratios later in the player record
| 0x6E + 0x24*i* | Team ID | uint8 | Value only relevant if `is teams` is true. 0 = red, 1 = blue, 2 = green
| 0x71 + 0x24*i* | Player Bitfield | uint8 | Bit 8 indicates the player has rumble enabled (0x80). All other bits unknown
| 0x74 + 0x24*i* | CPU Level | uint8 | Indicates what the CPU level is. Still specified on human players
| 0x79 + 0x24*i* | Offense Ratio | float | Indicates a knockback multiplier when this player hits another
| 0x7D + 0x24*i* | Defense Ratio | float | Indicates a knockback multiplier when this player is hit
| 0x81 + 0x24*i* | Model Scale | float | Indicates a multiplier on the size scaling of the character's model

### Frame Start
Frame start is an event added to transfer RNG seed at the very beginning of a frame's processing to prevent desyncs such as the knockback spiral animation desync.

| Offset | Name | Type | Description | Added
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x3A) The command byte for the frame start event | 2.2.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 2.2.0
| 0x5 | Random Seed | uint32 | The random seed at the start of the frame| 2.2.0

### Pre-Frame Update
This event will occur exactly once per frame per character (Ice Climbers are 2 characters). Contains information required to **reconstruct a replay**. Information is collected right before controller inputs are used to figure out the character's next action. The post-frame update contains the appropriate character ID to make sense of the action state present here.

| Offset | Name | Type | Description | Added
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x37) The command byte for the pre-frame update event | 0.1.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 0.1.0
| 0x5 | Player Index | uint8 | Between 0 and 3. Port is index + 1 | 0.1.0
| 0x6 | Is Follower | bool | Value is 1 for Nana and 0 otherwise | 0.1.0
| 0x7 | Random Seed | uint32 | The random seed at this point | 0.1.0
| 0xB | Action State ID | uint16 | Indicates the state the character is in. Very useful for stats | 0.1.0
| 0xD | X Position | float | X position of character | 0.1.0
| 0x11 | Y Position | float | Y position of character | 0.1.0
| 0x15 | Facing Direction | float | -1 if facing left, +1 if facing right (range: [-1, 1]) | 0.1.0
| 0x19 | Joystick X | float | Processed analog value of X axis of joystick (range: [-1, 1]) | 0.1.0
| 0x1D | Joystick Y | float | Processed analog value of Y axis of joystick (range: [-1, 1]) | 0.1.0
| 0x21 | C-Stick X | float | Processed analog value of X axis of c-stick (range: [-1, 1]) | 0.1.0
| 0x25 | C-Stick Y | float | Processed analog value of Y axis of c-stick (range: [-1, 1]) | 0.1.0
| 0x29 | Trigger | float | Processed analog value of trigger (range: [0, 1]) | 0.1.0
| 0x2D | Buttons | uint32 | Processed buttons. Look at bits set to see processed buttons pressed. Lower uint16 is equivalent to the `Physical Buttons` field. Upper uint16 has the following values: 0x10000 = Joystick Up, 0x20000 = Joystick Down, 0x40000 = Joystick Left, 0x80000 = Joystick Right, 0x100000 = C-Stick Up, 0x200000 = C-Stick Down, 0x400000 = C-Stick Left, 0x800000 = C-Stick Right, 0x80000000 = Trigger | 0.1.0
| 0x31 | Physical Buttons | uint16 | Use bits set to determine physical buttons pressed. Useful for APM. Bitwise mapping is `***S YXBA *LRZ UDRL`. `*` represents an unused bit | 0.1.0
| 0x33 | Physical L Trigger | float | Physical analog value of L trigger (range: [0, 1]). Useful for APM | 0.1.0
| 0x37 | Physical R Trigger | float | Physical analog value of R trigger (range: [0, 1]). Useful for APM | 0.1.0
| 0x3B | X analog for UCF | uint8 | Raw x analog controller input. Used by UCF dashback code | 1.2.0
| 0x3C | Percent | float | Current damage percent | 1.4.0

### Post-Frame Update
This event will occur exactly once per frame per character (Ice Climbers are 2 characters). Contains information for **making decisions about game states**, such as computing stats. Information is collected at the end of the Collision detection which is the last consideration of the game engine.

| Offset | Name | Type | Description | Added |
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x38) The command byte for the post-frame update event | 0.1.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 0.1.0
| 0x5 | Player Index | uint8 | Between 0 and 3. Port is index + 1 | 0.1.0
| 0x6 | Is Follower | bool | Value is 1 for Nana and 0 otherwise | 0.1.0
| 0x7 | Internal Character ID | uint8 | Internal character ID. Can only change throughout game for Zelda/Sheik. Before 1.6.0, check frame -123 to determine if Zelda started as Sheik. After 1.6.0, the game start character ID should be consistent with the started character | 0.1.0
| 0x8 | Action State ID | uint16 | Indicates the state the character is in. Very useful for stats | 0.1.0
| 0xA | X Position | float | X position of character | 0.1.0
| 0xE | Y Position | float | Y position of character | 0.1.0
| 0x12 | Facing Direction | float | -1 if facing left, +1 if facing right | 0.1.0
| 0x16 | Percent | float | Current damage percent | 0.1.0
| 0x1A | Shield Size | float | Current size of shield (range: [0, 60]) | 0.1.0
| 0x1E | Last Attack Landed | uint8 | ID of last attack that hit enemy | 0.1.0
| 0x1F | Current Combo Count | uint8 | The combo count as defined by the game | 0.1.0
| 0x20 | Last Hit By | uint8 | The player that last hit this player | 0.1.0
| 0x21 | Stocks Remaining | uint8 | Number of stocks remaining | 0.1.0
| 0x22 | Action State Frame Counter | float | Number of frames action state has been active. Can have a fractional component for certain actions | 0.2.0
| 0x26 | State Bit Flags 1 | 8 bits | Bits indicate state flags. Known bits: `0x10 = isReflectActive` | 2.0.0
| 0x27 | State Bit Flags 2 | 8 bits | Bits indicate state flags. Known bits: `0x04 = HasIntangOrInvinc` `0x08 = isFastFalling` `0x20 = isHitlag` | 2.0.0
| 0x28 | State Bit Flags 3 | 8 bits | Bits indicate state flags. Known bits: `0x80 = isShieldActive` | 2.0.0
| 0x29 | State Bit Flags 4 | 8 bits | Bits indicate state flags. Known bits: `0x2 = isHitstun` `0x4 = owners detection hitbox touching shield bubble` `0x20 = Powershield Active Bool` | 2.0.0
| 0x2A | State Bit Flags 5 | 8 bits | Bits indicate state flags. Known bits: `0x80 = isOffscreen` `0x40 = isDead` `0x10 = inSleep` `0x8 = isFollower` | 2.0.0
| 0x2B | Misc AS (Hitstun remaining) | float | Can be used for different things. While isHitstun true, contains hitstun frames remaining | 2.0.0
| 0x2F | Ground/Air State | bool | 0 if grounded, 1 is airborne | 2.0.0
| 0x30 | Last Ground ID | uint16 | ID of the last ground the character stood on | 2.0.0
| 0x32 | Jumps Remaining | uint8 | Number of jumps remaining | 2.0.0
| 0x33 | L-Cancel Status | uint8 | 0 = none, 1 = successful, 2 = unsuccessful | 2.0.0

### Item Update
A maximum of 15 items per frame can have their data extracted. This information can be used for stats, training AIs, or visualization engines to handle items. Note that these aren't just for "items" it also includes all projectiles such as Fox lasers, Sheik needles, etc.

| Offset | Name | Type | Description | Added
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x3B) The command byte for the item update event | 3.0.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 3.0.0
| 0x5 | Type ID | uint16 | The type of item this is | 3.0.0
| 0x7 | State | uint8 | The state the item is in. Mostly undocumented, might differ per type | 3.0.0
| 0x8 | Facing Direction | float | -1 if facing left, +1 if facing right | 3.0.0
| 0xC | X Velocity | float | X velocity of item | 3.0.0
| 0x10 | Y Velocity | float | Y velocity of item | 3.0.0
| 0x14 | X Position | float | X position of item | 3.0.0
| 0x18 | Y Position | float | Y position of item | 3.0.0
| 0x1C | Damage Taken | uint16 | Amount of damage an item has taken | 3.0.0
| 0x1E | Expiration Timer | float | Number of frames remaining before item expires. Can go into the negatives for certain items such as link arrows | 3.0.0
| 0x22 | Spawn ID | uint32 | Auto-incremented number whenever an item spawns: 0, 1, 2, 3, etc | 3.0.0

### Frame Bookend
The frame bookend is a simple event that can be used to determine that the entire frame's worth of data has been transferred/processed. It is always sent at the very end of the frame's transfer.

| Offset | Name | Type | Description | Added
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x3C) The command byte for the frame bookend event | 3.0.0
| 0x1 | Frame Number | int32 | The number of the frame. Starts at -123. Frame 0 is when the timer starts counting down | 3.0.0

### Game End
This event indicates the end of the game has occurred. If present, this will occur exactly once in the stream. It may not be present.

| Offset | Name | Type | Description | Added
| --- | --- | --- | --- | --- |
| 0x0 | Command Byte | uint8 | (0x39) The command byte for the game end event | 0.1.0
| 0x1 | Game End Method | uint8 | See following table | 0.1.0
| 0x2 | LRAS Initiator | int8 | Index of player that LRAS'd. -1 if not applicable | 2.0.0

#### Game End Method
The behavior of this field depends on the version.

| Version | Values |
| --- | --- |
| 0.1.0 | 0 = Unresolved, 3 = resolved
| 2.0.0 | 1 = TIME!, 2 = GAME!, 7 = No Contest

# The metadata element
The metadata element contains any miscellaneous data relevant to the game but not directly provided by Melee. Unlike all the other data defined in this doc, which was basically stored as a binary stream, the data in the metadata element is pure UBJSON.

The metadata element can be read individually to save processing time with a bit of effort. For a complete file, the raw element will indicate its size, meaning the entire data block can be skipped in order to extract just the metadata element.

Key | Type | Description |
| --- | --- | --- |
| startAt | string | Timestamp of when the game started, ISO 8601 format (e.g. `2018-06-22T07:52:59Z`)
| lastFrame | int32 | The frame number of the last frame of the game. Used to show game duration without parsing entire replay
| players | object | Metadata for the individual players. Currently just contains character-use statistics (example: `{'characters': {'18': 5209}}` means the player used Marth (internal character ID `18`) for all `5209` frames of the game. Mostly useful for Zelda/Sheik
| playedOn | string | Platform the game was played on (values include `dolphin`, `console`, and `network`)
| consoleNick | string | The name of the console the replay was created on.
