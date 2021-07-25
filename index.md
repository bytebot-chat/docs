## Bytebot

Bytebot is more than a chatbot. It's a lightweight framework that allows you to connect anything to anything. Slack, RSS, Twitter, or your own protocols and gateways -- Bytebot lets you talk to protocols without worrying about the details. You write your business logic and let the router handle the rest.

Features:
 - Multiple protocol gateways
 - Easy to write apps
 - Standardized messaging format
 - Compatible with any language
 - Designed for Kubernetes and cloud infrastructure first
 - Simple deployment through helm, docker, docker-compose, or binaries

### Quickstart

Ready to jump in before you've read everything? Follow our quickstart guide to get Bytebot up and running fast.

[Start the tour](quickstart.md)

### Simplicity and minimalism are built right in

Apps for Bytebot require a minimum of boilerplate or configuration. They need only to be able to hold a TCP socket open and read/write JSON to that socket. This means you can write in virtually any language.

Messages follow a common, consistent format for each protocol and overlap common fields when possible, meaning you can subscribe to and handle multiple protocols with ease.

Because messages from gateways are merely JSON objects, your app does not need to be aware of how a protocol works in order to interact with it. To have a discord bot, you don't need to know how to open the Discord websocket, authenticate, or maintain a connection. You merely need to subscribe to messages originating from the Bytebot discord gateway.

For example, here is a fully working app that leverages the discord gateway

```
package main

import (
	"context"
	"flag"
	"fmt"
	"time"

	"github.com/bytebot-chat/gateway-discord/model"
	"github.com/go-redis/redis/v8"
	"github.com/satori/go.uuid"
)

var addr = flag.String("redis", "localhost:6379", "Redis server address")
var inbound = flag.String("inbound", "discord-inbound", "Pubsub queue to listen for new messages")
var outbound = flag.String("outbound", "discord", "Pubsub queue for sending messages outbound")

func main() {
	flag.Parse()
	ctx := context.Background()

	rdb := redis.NewClient(&redis.Options{
		Addr: *addr,
		DB:   0,
	})

	err := rdb.Ping(ctx).Err()
	if err != nil {
		time.Sleep(3 * time.Second)
		err := rdb.Ping(ctx).Err()
		if err != nil {
			panic(err)
		}
	}

	topic := rdb.Subscribe(ctx, *inbound)
	channel := topic.Channel()
	for msg := range channel {
		m := &model.Message{}
		err := m.Unmarshal([]byte(msg.Payload))
		if err != nil {
			fmt.Println(err)
		}
		if m.Content == "ping" {
			reply(ctx, *m, rdb)
		}
	}
}

func reply(ctx context.Context, m model.Message, rdb *redis.Client) {
	metadata := model.Metadata{
		Dest:   m.Metadata.Source,
		Source: "discord-pingpong",
		ID:     uuid.Must(uuid.NewV4(), *new(error)),
	}
	stringMsg, _ := m.MarshalReply(metadata, m.ChannelID, "pong")
	rdb.Publish(ctx, *outbound, stringMsg)
	return
}
```

### Don't change everything, just update your apps

Have you noticed that with virtually every chatbot, updating a single piece of the bot requires rebuilding and redeploying the entire thing? With bytebot, every app runs independently of every other one, including the gateway. Updating your service is seamless and transparent to the user. They never even notice a reconnect.

### Connecting lots of servers is a breeze

Want your app to talk to lots of endpoints at once? With bytebot, you can run a single instance of the application and multiple gateways. You now have a single bot with presence on an arbitrary number of servers, which can be challenging in many protocols.

### Bridging and interacting with multiple protocols is trivial

Want to send IRC messages to discord? Read RSS and publish to twitter? Write a message broadcast system that hits dozens of endpoints or channels at once? Ingest messages from hundreds of different sources? Gateways ingest events, serialize them to JSON, and publish them as they get them. Your app only needs to be able to deserialize the JSON and handle it however it chooses. It doesn't need to know multiple protocols to speak to multiple protocols.
