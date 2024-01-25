# Welcome to Bytebot Documentation

Bytebot is a versatile messaging solution designed to efficiently route and process messages across different platforms. Comprising a lightweight message gateway, router, and extensible subscribers, Bytebot offers flexibility for diverse messaging needs.

## Key Features

- **Message Routing:** Bytebot efficiently routes messages from various sources, providing a centralized solution for message processing.

- **Extensibility:** Develop custom subscribers in any language to integrate with message gateways, tailoring functionality to your specific requirements.

- **Redis Pubsub Integration:** Leveraging Redis Pubsub, Bytebot ensures seamless communication between components, allowing for scalable and real-time message handling while decoupling consumers from publishers.

## How It Works

This system has three components: a message router, a message gateway, and a message consumer. 

Below is an example using two subscribers listening for messages from Discord.

![Flow of messages through bytebot's pubsub system][messageFlow]

[messageFlow]: https://mermaid.ink/img/pako:eNqNks1uwjAMx1_Fijgh4AE6aRLQgThsmsZuDYe08WhEm1T52IYo7z6XUMY2aaLpIXHs_8-xfWCFkcgStrWiKeE1veOa-2mWKlcYK8GhfUe7gfH4HtqVzk3QEmp0TmzRwZs1NZxdW5hlj_ECtsLjh9hvTmKzGAzPIa-UK_to8Ib-RhUJ5xpUlJ7IKDYZRPAqnQyKUmiNVbcNrjO1MM9ekDw7SRfyDgP0zc-gNVkLqxqvjIZGeI9WA34WVZCU84lJyCfje88cJYU9ZK4_2uRvQsaXaNd9VsOboEpfQ7uHXhEpKP0XeWGkvyposan2YDTpmeBvLxxJLM6Va2Llov7i5xu65Kg5f6SH0C4vLV6eWxwVlr0Cfo8H5KLYdUIxI2inNA202IjVaGuhJI3doYvnjGpbI2cJbaWwO864PpKfCN6s97pgibcBRyw0krCpEjStdTQevwA-9PUM?type=png

### Getting Started

Once you've had a look at the [architecture](#architecture) and [key concepts](#key-concepts), you can follow the [quickstart guide](#quickstart-guide) to get started.

## Architecture

Bytebot is comprised of three components: a message router, a message gateway, and a message consumer.

### Message Gateway

The message gateway is responsible for sending and receiving messages from a specific platform or protocol. For example, the Discord message gateway is responsible for receiving messages from discord and serializing them into a format that can be understood by the message broker and comsumers. It is also responsible for deserializing messages from the message router and sending them to the correct channel on Discord.

Message gateways should ideally be stateless, and should not be responsible for any message processing. Instead, they should simply pass messages to the message broker for processing.

### Message Broker

The message broker is responsible for receiving messages from the message gateway and routing them to the correct message consumer. It is also responsible for receiving messages from the message consumer and routing them to the correct message gateway.

This is handled entirely by Redis Pubsub, which allows for scalable and real-time message handling while decoupling consumers from publishers.

### Message Consumer

The message consumer is responsible for receiving messages from the message broker and processing them. This can be done in any language, and can be as simple or complex as you need it to be.

Return messages are sent back to the message broker, which then routes them to the correct message gateway.

## Key Concepts

### Topics

Topics are the primary means of routing messages in Bytebot. They are hierarchical in nature, and are used to route messages to the correct message consumer.

Topics may be protocol specific, such as `discord` or `telegram`, or they may be generic, such as `inbound` or `outbound`.

Example: `inbound.discord.1190804811990446190.1200058256127688735.179258058118135808` represents a message received from Discord, and contains the following information:

- `inbound`: The message is inbound, meaning it was received from a message gateway.
- `discord`: The message was received from the Discord message gateway.
- `1190804811990446190`: The ID of the guild the message was received from.
- `1200058256127688735`: The ID of the channel the message was received from.
- `179258058118135808`: The ID of the user who sent the message.

This topic structure allows for easy routing of messages to the correct message consumer or subscribers to choose which messages they want to receive. The could be as simple as a subscriber that only listens for messages from a specific channel, or as complex as a subscriber that listens for messages from a specific user in a specific channel in a specific guild. 

Additionally, Redis allows clients to subscribe to multiple topics at once using `PSUBSCRIBE`, which allows for efficient message handling, and use multiple patterns. For example, a subscriber could listen for messages from all channels in a specific guild by subscribing to `inbound.discord.1190804811990446190.*.*.*` or all messages from a specific user by subscribing to `inbound.discord.*.*.*.179258058118135808`.

### Subscribers

Subscribers are responsible for receiving messages from the message broker and processing them. They can be written in any language, and can be as simple or complex as you need them to be.

Subscribers are just standalone programs that connect to the Redis server and subscribe to the topics they are interested in. They are responsible for their own serialization and deserialization of messages and are not limited exclusively to Bytebot. For example, you could write a subscriber that listens for messages from a specific channel and sends them to a database or takes action in a game based on the message content.

Return messages are sent back to the message broker, which then routes them to the correct message gateway.