## Writing your first app

Even though we have a working gateway, we still don't have anything to respond to events. Let's change that by writing a simple program that responds to a message containing "ping" with the word "pong."

In the project directory you created previously, add a new directory called `pingpong` and navigate into it.

First, let's create a new file called `pingpong.go` and populate it with the following content:

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

### Compile and build
We _could_ compile and run it here with `go run ./main.go`, but let's add a Dockerfile and incorporate it into our application stack so we can keep things consistent.

Create a new file called `Dockerfile` and populate it with the following content:

```
FROM golang:1.16.4-alpine3.13 as builder

RUN adduser -D -g 'bytebot' bytebot
WORKDIR /app
COPY . .
RUN apk add --no-cache git tzdata
RUN GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -a -ldflags "-s -w -extldflags '-static'" -o ./opt/bytebot

FROM scratch
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/group /etc/group
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /app/opt/bytebot /opt/bytebot

USER bytebot
ENTRYPOINT ["/opt/bytebot"]
```

Now with a working Dockerfile, we can `docker build` our application. Next, we will incorporate it into Docker Compose to make building and rebuilding easy. Update your `docker-compose.yaml` from the previous step to include your new app:

```
version: "3.8"
services:
  bytebot:
    image: dkr.fraq.dev/bytebot/gateway-discord:edge
    command:
      - "-id"
      - "discord"
      - "-inbound"
      - "discord-inbound"
      - "-outbound"
      - "-discord-outbound"
      - "-token"
      - "your-discord-bot-token-goes-here"
      - "-redis"
      - "redis:6379"
  redis:
    image: redis:6.2.3
    ports:
      # So we can connect and watch messages on the queue if we need to
      - "127.0.0.1:6379:6379"
  pingpong:
    build: ./pingpong
    command:
      - "-inbound"
      - "discord-inbound"
      - "-outbound"
      - "discord-outbound"
      - "-redis"
      - "redis:6379"
```

Now run `docker-compose up -d pingpong` from your project directory.

You can attach to the container and view output the same way you did with the gateway by running `docker-compose logs -f pingpong`.

At this point, your bot should be responding to messages. Try it out!

#### Next
- [Message routing](message-routing.md)

#### Previous
- [Quickstart](quickstart.md)
