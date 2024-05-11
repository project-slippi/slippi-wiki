# Transport Layer
The spectator communications from Dolphin use ENet connections, by default listening on UDP port 51441.

So the networking layers look something like this:

```
======
 IP
======
 UDP
======
 ENet
======
 Application
======
```

# ENet Layer
To connect to Slippi's Dolphin spectator server, send an ENet connection and then follow the application layer protocol described below.

The ENet layer itself is otherwise fairly uneventful. A single reliable channel is used.

# Application Layer
The application layer protocol takes place entirely using JSON and is *mostly* stateless. The first exception occurs right off the bat with a handshake message. This is initiated by the client, and is formatted as follows:

## Handshake

```
{
  "type": "connect_request",
  "cursor": INTEGER
}
```

This message must be sent first, and no streaming data will be sent by the server until it is. The "cursor" field specifies which message in the stream to start from. (0 to start) So if you would like to start receiving message from the start of the match, set the cursor to 0. If you are picking back up from an interrupted stream in the middle, then specify the cursor where you left off.

The server will respond with a connect response formatted as follows:

```
{
  "type": "connect_reply",
  "nick": "SOME_STRING",  
  "version": "SLIPPI_SEMVER_STRING",
  "cursor": INTEGER
}
```

| Value | Description |
| ---- | ----------- |
| nick | The nickname of the console. This can be set to most any string and is meant to (mostly) uniquely identify the console when there are several in the room. |
| version | The semantic version of the Slippi running on the server, in string form. Such as "2.2.1". |
| cursor | The index that the server will start sending messages from. *This may not be the cursor you requested*. This is for a good reason, the cursor you asked for may not be valid anymore, for instance. |

This can commonly happen when you try to rejoin a match that is no longer in progress for instance.

## SLP Streams
After having finished a handshake with the server, you will now receive streaming Slippi event messages.

These can take one of two forms, with very similar formats. Game Events and Menu Events.

### Game Events

Game events will arrive formatted as such:

```
{
  "type": "game_event",
  "cursor": INTEGER,
  "next_cursor": INTEGER,
  "payload": "BASE64_ENCODED_BINARY"
}
```

Notably, each "event" here is a batch of SLP events concatenated together. In practice, they will be batched together as a frame though this shouldn't be assumed for the purpose of parsing.


| Value | Description |
| ---- | ----------- |
| cursor | The integer cursor of this message. |
| next_cursor | The cursor to expect next. In practice, this is monotonically increasing but clients are advised to respect the "next_cursor" index as this might not always be the case. |
| payload | A base64-encoded binary string of the actual Slippi events.|

### Menu Events

Menu events represent things that occur at the various Melee menus which are not captured by the SLP file format. These events are useful for bots in particular, so that the bot is able to do things like pick its own character. But the information here is generally useful too.

Menu events are formatted as follows:

```
{
  "type": "menu_event",
  "payload": "BASE64_ENCODED_BINARY"
}
```

Note that menu events do not have a cursor. They operate in a "use it or lose it" environment that provides a snapshot of the current menu environment, but no history.

Menu events are created and sent once per frame.
