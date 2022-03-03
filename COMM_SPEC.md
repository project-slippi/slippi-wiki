# Intro
This document will outline the details of the Slippi Dolphin communication file (.json). This file is used to direct a Slippi Playback Dolphin to the correct Slippi replay file. Dolphin can be loaded with a json by launching it like so: `<dolphin binary> -i <json path>`. The minimum viable json for playback is `{"replay": "<path to replay>"}`.

## Top Level

| Name | Type | Description |
| --- | --- | --- |
| mode | string | Possible values are `normal` (default), `queue`, and `mirror` |
| replay | string | The path to the replay if in normal or mirror mode |
| startFrame | int | The frame you would like to start the replay on, default is `-123` |
| endFrame | int | The frame you would like to end the replay on, default is `INT_MAX` |
| commandId | string | Typically used to indicate that the replay has changed, but updating the value can also restart playback of the current replay or queue |
| outputOverlayFiles | boolean | Will output the console name and time of replay to the `Slippi` folder next to the Dolphin executable (this only works when using `queue` mode), default is `false` |
| isRealTimeMode | boolean | Will force dolphin to stay closer to realtime which is important for mirroring, default is `false` |
| shouldResync | boolean | Indicates whether the resync logic should be used. Resync logic will allow playback to go back to normal after a desync, default is `true` |
| rollbackDisplayMethod | string | Tells dolphin to display rollbacks either like the player saw them (`normal`) or by showing every frame in the file (`visible`). Possible values are `off` (default), `normal`, and `visible`. |
| queue | QueueItem[] | All files in the queue will be played back to back. This is commonly used for set recordings or combo video recordings |

## QueueItem

| Name | Type | Description |
| --- | --- | --- |
| path | string | the path to the replay |
| startFrame | int | The frame you would like to start the replay on, default is `-123` |
| endFrame | int | The frame you would like to end the replay on, default is `INT_MAX` |
| gameStartAt | string | Typically the time of the replay, but can be used with any string |
| gameStation | string | Typically the name of console the replay was created on, but can be used with any string |

Additional data in the JSON will not interfere with playback as long as the JSON continues to be valid.

## Example
```
{
	"mode":"normal",
	"rollbackDisplayMethod":"off",
	"shouldResync":false,
	"replay": "C:\\Path\\Replay.slp",
	"commandId": "1"
}
```
