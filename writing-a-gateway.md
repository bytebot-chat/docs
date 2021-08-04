# Writing a gateway

### Overview

Simply put, a gateway gets information from a plaftorm, encapsulate this information into
a Message, adds routing metadata, and puts it on the outbound queue of a redis database.
It also reads messages from an inbound queue and pushes them towards the platform.

By convention, a gateway for a platform is named `gateway-{platform name}`

### Writing a gateway

We are going to go over gateway-rss as an example.
It's a simple readonly gateway, so we won't use the inbound queue.

The code is partial, take a look at [gateway-rss](https://github.com/bytebot-chat/gateway-rss) for it's complete version.

#### Data structure

The only data structure that matters is the Message struct.
We are using a [library to handle rss](https://github.com/SlyMarbo/rss), so
we will extend it's main structure:

```golang
// Message and related utils, this is the base unit.
type Message struct {
	*rss.Item
	Metadata Metadata
}

// This is standard over _all_ gateways.
type Metadata struct {
	Source string
	Dest   string
	ID     uuid.UUID
}

func (m *Message) Marshal() ([]byte, error) {
	return json.Marshal(m)
}

func MessageFromItem(item *rss.Item) *Message {
	msg := new(Message)
	msg.Item = item

	return msg
}
```

Please note that the Metadata struct must be followed exactly for apps to be able to parse them.


### Reading a feed and pushing it's feeds to the queue

We are extending the `Feed` struct from our rss handling library:

```golang
/// Feed struct and utils, this describes the feed we follow.
type Feed struct {
	*rss.Feed
}

// Reading new items and pushing it to the queue.
func pushNewItemsToQueue(feed *rss.Feed) error {
	for _, i := range feed.Items {
		if !i.Read {
			msg := MessageFromItem(i)
			msg.Metadata.ID = uuid.Must(uuid.NewV4(), *new(error))
			msg.Metadata.Source = feedURL
			msg.Metadata.Dest = "gateway-rss"

			stringMsg, _ := json.Marshal(msg)
			rdb.Publish(ctx, inbound, stringMsg)

			i.Read = true
		}
	}
	return nil
}

```

The pushNewItemsToQueue method is an abstraction that we reuse in the main loop.

### The main function
The main function is pretty straightforward, first connect to redis and read the feed
then refresh the feed and push it's new items to redis.

```
func main() {
	rdb = rdbConnect("127.0.0.1:6379")
	ctx = context.Background()
	delay = 3

	feed, _ := rss.Fetch(feedURL)

	for {
		feed.Refresh = time.Now()
		feed.Update()
		pushNewItemsToQueue(feed)

		time.Sleep(time.Second * delay)
	}
}
```

### Putting it all together

Here is the full gateway:

```golang
package main

import (
	"context"
	"encoding/json"
	"time"

	"github.com/SlyMarbo/rss"
	"github.com/go-redis/redis/v8"
	"github.com/satori/go.uuid"
)

var (
	ctx context.Context
	rdb *redis.Client

	delay time.Duration

	feedURL   = "http://localhost:8000/flux.xml"
	redisAddr = "127.0.0.1:6379"
	inbound   = "rss-inbound"
	// we don't need an outbound queue for a rss gateway
)

func main() {
	rdb = rdbConnect("127.0.0.1:6379")
	ctx = context.Background()
	delay = 3

	feed, _ := rss.Fetch(feedURL)

	for {
		feed.Refresh = time.Now()
		feed.Update()
		pushNewItemsToQueue(feed)

		time.Sleep(time.Second * delay)
	}
}

/// Feed struct and utils, this describes the feed we follow.
type Feed struct {
	*rss.Feed
}

// Reading new items and pushing it to the queue.
func pushNewItemsToQueue(feed *rss.Feed) error {
	for _, i := range feed.Items {
		if !i.Read {
			msg := MessageFromItem(i)
			msg.Metadata.ID = uuid.Must(uuid.NewV4(), *new(error))
			msg.Metadata.Source = feedURL
			msg.Metadata.Dest = "gateway-rss"

			stringMsg, _ := json.Marshal(msg)
			rdb.Publish(ctx, inbound, stringMsg)

			i.Read = true
		}
	}
	return nil
}

// Message and related utils, this is the base unit.
type Message struct {
	*rss.Item
	Metadata Metadata
}

// This is standard over _all_ gateways.
type Metadata struct {
	Source string
	Dest   string
	ID     uuid.UUID
}

func (m *Message) Marshal() ([]byte, error) {
	return json.Marshal(m)
}

func MessageFromItem(item *rss.Item) *Message {
	msg := new(Message)
	msg.Item = item

	return msg
}

// Redis connection function
func rdbConnect(addr string) *redis.Client {
	rdb := redis.NewClient(&redis.Options{
		Addr:     addr,
		Password: "", // no password set
		DB:       0,  // use default DB
	})

	return rdb
}
```

This is very basic, it lacks logging and configuration, for starters.

Take a look at [gateway-rss](https://github.com/bytebot-chat/gateway-rss) if you need to pipe your rss feeds into bytebot.

### Deploying it

Let's use docker-compose:

```Dockerfile
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

# Our chosen default for Prometheus
USER bytebot
ENTRYPOINT ["/opt/bytebot"]
```

and the docker-compose:

```yaml
version: "3.8"
services:
  bytebot:
    build: .
  redis:
    image: redis:6.2.3
    ports:
      - "127.0.0.1:6379:6379"
```

Run it with `docker-compose -d` and voilà
