# Spectate Remote Control
Spectate Remote Control allows a user to programmatically control Slippi Launcher's spectate feature via WebSocket.
This is most useful for streaming netplay tournaments.

## API
You must connect to Spectate Remote Control from the same computer.
Spectate Remote Control allows only 1 connection at a time.

### Protocol
Spectate Remote Control accepts only one protocol: `spectate-protocol`
```
> let socket = new WebSocket("ws://localhost:49809", "spectate-protocol");
> socket.onopen = () => { console.log("open") };
> socket.onmessage = (event) => { console.log("message:\n" + event.data) };
> socket.onerror = (event) => { console.log("error: " + JSON.stringify(event)) };
> socket.onclose = () => { console.log("close") };
open
```

### Events
Upon connecting, Spectate Remote Control will always send `spectating-broadcasts-event` once to tell the client about the current state:
```
message:
{
  "op":"spectating-broadcasts-event",
  "spectatingBroadcasts": [
    {
      "broadcastId":"q0ejrrUneJcA5v65nUVX13OYr0o2-oFTsLA8jxfP9GqqVt7kxYk",
      "dolphinId":"1713518126142broadcast0"
    }
  ]
}
```

The following events may be sent at any time
```
message:
{
  "op":"new-file-event",
  "broadcastId":"q0ejrrUneJcA5v65nUVX13OYr0o2-oFTsLA8jxfP9GqqVt7kxYk",
  "dolphinId":"spectate-q0ejrrUneJcA5v65nUVX13OYr0o2-cYp8HZJ9FB3P3uW72b2dHM",
  "filePath":"C:\\Users\\jmlee337\\Documents\\Slippi\\Spectate\\Game_20240309T104448.slp"
}
```
```
message:
{
  "op":"game-end-event",
  "broadcastId":"q0ejrrUneJcA5v65nUVX13OYr0o2-oFTsLA8jxfP9GqqVt7kxYk",
  "dolphinId":"spectate-q0ejrrUneJcA5v65nUVX13OYr0o2-cYp8HZJ9FB3P3uW72b2dHM"
}
```
```
message:
{
  "op":"dolphin-closed-event",
  "dolphinId":"spectate-q0ejrrUneJcA5v65nUVX13OYr0o2-cYp8HZJ9FB3P3uW72b2dHM"
}
```

### Requests
A client may send the following requests:

#### List Broadcasts
```
> socket.send(JSON.stringify({op: "list-broadcasts-request"}))
message:
{
  "op":"list-broadcasts-response",
  "broadcasts": [
    {
      "id":"q0ejrrUneJcA5v65nUVX13OYr0o2-cYp8HZJ9FB3P3uW72b2dHM",
      "name":"LOST#882",
      "broadcaster": {
        "uid":"q0ejrrUneJcA5v65nUVX13OYr0o2",
        "name":"lostletters"
      }
    }
  ]
}
```

#### Spectate Broadcast
If `dolphinId` is not set, spectating will begin in a new window with a generated `dolphinId`:
```
> socket.send(JSON.stringify({op: "spectate-broadcast-request", broadcastId: "PMmdJgqjWzWrMHmq2IbktBKpbcf2-4zXtnoxyBgA2jRwH6wYbtb"}));
message:
{
  "op":"spectate-broadcast-response",
  "broadcastId":"q0ejrrUneJcA5v65nUVX13OYr0o2-oFTsLA8jxfP9GqqVt7kxYk",
  "dolphinId":"1710210546025remote0",
  "path":"C:\\Users\\jmlee337\\Documents\\Slippi\\Spectate"
}
```
If `dolphinId` is set, spectating will begin in the corresponding dolphin window if one exists, or a new one if it doesn't:
```
> socket.send(JSON.stringify({op: "spectate-broadcast-request", broadcastId: "PMmdJgqjWzWrMHmq2IbktBKpbcf2-4zXtnoxyBgA2jRwH6wYbtb", dolphinId: "dolphin-id"}));
message:
{
  "op":"spectate-broadcast-response",
  "broadcastId":"q0ejrrUneJcA5v65nUVX13OYr0o2-oFTsLA8jxfP9GqqVt7kxYk",
  "dolphinId":"dolphin-id",
  "path":"C:\\Users\\jmlee337\\Documents\\Slippi\\Spectate"
}
```

### Request Errors
Request errors are denoted by the presence of an `err` field in the response:
```
> socket.send(JSON.stringify({op: "spectate-broadcast-request"}));
message:
{
  "op":"spectate-broadcast-response",
  "err":"no broadcastId"
]
```
