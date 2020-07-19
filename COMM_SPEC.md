# Intro
This document will outline the details of the Slippi Dolphin communication file (.json). This file is used to direct a Slippi Playback Dolphin to the correct Slippi replay file. Dolphin can be loaded with a json by launching it like so: `<dolphin binary> -i <json path>`.

## Top Level JSON

| Name | Type | Description |
| --- | --- | --- |
| mode | string | commonly used values are `normal`, `queue`, and `mirror` |
| replay | string | the path to the replay if in normal or mirror mode |
| startFrame | int | the frame you would like to start the replay on, default is `-123` |
| endFrame | int | the frame you would like to end the replay on, default is `INT_MAX` |
| commandId | string | typically used to indicate that the replay has changed, but updating the value can also restart playback of the current replay or queue |
| outputOverlayFiles | boolean | will output the console name and time of replay to the `Slippi` folder next to the Dolphin executable |
| isRealTimeMode | boolean | will force dolphin to stay closer to realtime which is important for mirroring |
| queue | QueueItem[] | all files in the queue will be played back to back. This is commonly used for set recordings or combo video recordings |

## QueueItem

| Name | Type | Description |
| --- | --- | --- |
| path | string | the path to the replay |
| startFrame | int | the frame you would like to start the replay on, default is `-123` |
| endFrame | int | the frame you would like to end the replay on, default is `INT_MAX` |
| gameStartAt | string | any string is fine, but it is typically for the time of the replay |
| gameStation | string | any string is fine, but it is typically for the name of console the replay was created on |

Additional data in the JSON will not interfere with playback as long as the JSON continues to be valid.
