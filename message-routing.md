## How messages are routed

### Consuming messages

When you receive message from the subscribed Redis queue, the redis package for you language of choice should return an object of key:value stores (called a struct in Go, dictionary in Python, etc.). The message will be in the following format.

```json
{
  "channel": "discord-inbound", // The Redis channel the message came from
  "data": "{\"id\":\"1025070959729844254...\"}", // Message data (stringified JSON)
  "pattern": null,
  "type": "message" // The protocol specific message type
}
```

The message data contains the message content and metadata from the protocol specific gateway. Below is an example message from Discord.

```json
{
  "Metadata": {
    "Dest": "",
    "ID": "cea0ffc6-15c7-49b0-9131-59a8e8640db0", // UID
    "Source": "discord" // The bytebot gateway ID
  },
  "activity": null,
  "application": null,
  "attachments": [],
  "author": {
    "avatar": "zyxw",
    "bot": false,
    "discriminator": "7649",
    "email": "",
    "flags": 0,
    "id": "0000000000000000000",
    "locale": "",
    "mfa_enabled": false,
    "premium_type": 0,
    "public_flags": 0,
    "system": false,
    "token": "",
    "username": "totally not a bot", // User who posted the message
    "verified": false
  },
  "channel_id": "123456789987654321", // Discord channel ID where the message was posted
  "content": "long live bytebot", // Content of the message
  "edited_timestamp": "",
  "embeds": [],
  "flags": 0,
  "guild_id": "135798642135798642",
  "id": "987654321123456789", // Discord message ID
  "member": {
    "deaf": false,
    "guild_id": "",
    "joined_at": "2018-03-31T18:12:55.578000+00:00",
    "mute": false,
    "nick": "",
    "pending": false,
    "premium_since": "",
    "roles": [],
    "user": null
  },
  "mention_channels": null,
  "mention_everyone": false,
  "mention_roles": [],
  "mentions": [],
  "message_reference": null,
  "pinned": false,
  "reactions": null,
  "timestamp": "2022-09-29T18:14:20.028000+00:00",
  "tts": false,
  "type": 0,
  "webhook_id": ""
}
```

### Reply to Messages

To reply to a message you will need to publish to the channel in Redis that the gateway is listening to, using the Redis package of your language of choice. The format of the message should be a JSON object. Below is an example for Discord.

```json
{
  "Content": "bytebot will live on forever", // What gets sent to the chat
  "From": "",
  "Metadata": {
    "Dest": "discord", // The bytebot gateway ID
    "ID": "98b9b6c0-1cb8-4ccf-9740-6ceb4c81515a", // UID (you should generate this)
    "Source": "MyApp" // The name of your app
  },
  "channel_id": "123456789987654321" // Discord Channel ID (can get this from the source message)
}
```

#### Next

- [Writing a gateway](writing-a-gateway.md)

#### Previous

- [Writing your first app](writing-your-first-app.md)

```

```
