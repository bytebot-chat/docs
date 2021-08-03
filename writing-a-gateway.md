# Writing a gateway

### Overview

Simply put, a gateway gets information from a plaftorm, encapsulate this information into
a Message, adds routing metadata, and puts it on the outbound queue of a redis database.
It also reads messages from an inbound queue and pushes them towards the platform.

By convention, a gateway for a platform is named `gateway-{platform name}`

### Message structure and metadata

A message must be exported to json. Along of the message's content, it's metadata
must contain three fields, "Source", "Dest", and "ID".

Once JSON encoded, a typical message looks like this:

```
//TODO
```

### Writing a gateway

We are going to go over gateway-rss as an example.
It's a simple readonly gateway, so we won't use the inbound queue.

The code is partial, take a look at [gateway-rss](https://github.com/bytebot-chat/gateway-rss) for it's complete version.

#### Connection to redis

We need to connect to a redis database, we will do so using [go-redis](https://github.com/go-redis/redis/v8)

```
// This function connects to a redis database and returns a client
func rdbConnect(addr string) *redis.Client {
	ctx := context.Background()
	log.Info().
		Str("redis", addr).
		Msg("connecting to redis...")
	rdb := redis.NewClient(&redis.Options{
		Addr:     addr,
		Password: "", // no password set
		DB:       0,  // use default DB
	})

	err := rdb.Ping(ctx).Err()
	if err != nil {
		time.Sleep(3 * time.Second)
		err := rdb.Ping(ctx).Err()
		if err != nil {
			log.Fatal().
				Err(err).
				Msg("Couldn't connect to redis")
			os.Exit(1)
		}
	}

	return rdb
}
```

#### Data structure

The only data structure that matters is the Message struct.
We are using a [library to handle rss](https://github.com/SlyMarbo/rss), so
we will extend it's main structure:

```golang
type Message struct {
	*rss.Item
	Metadata Metadata
}

type Metadata struct {
	Source string
	Dest   string
	ID     uuid.UUID
}

// Let's add a Marshal method to get it in proper json to push it:
func (m *Message) Marshal() ([]byte, error) {
	return json.Marshal(m)
}
```

### Reading a feed and pushing it's feeds to the queue
We are not going to go over the specifics of rss parsing, as this is rather uninteresting, but the following is the program's main function:

```golang
func main() {
	feed, _ := model.CreateFeed(feedURL)
	rdb = rdbConnect(redisAddr)
	ctx = context.Background()

	// Until the gateway is stopped, read new items on the feed,
	// transform them into messages, add metadata and push them on the queue.
	for {
		feed.Update()
		for _, i := range feed.Items {
			if !i.Read {
				msg := MessageFromItem(i)
				msg.Metadata.ID = uuid.Must(uuid.NewV4(), *new(error))
				msg.Metadata.Source = f.UpdateURL
				msg.Metadata.Dest = "gateway-rss"

				stringMsg, _ := json.Marshal(msg)
				rdb.Publish(ctx, inbound, stringMsg)

				i.Read = true
			}
		}
	
		time.Sleep(time.Second * delay)
	}
}
```

### Deploying a gateway

Now that we wrote this gateway, we need to deploy it, let's do so with docker-compose:

```Dockerfile
FROM golang:1.16.4-alpine3.13 as builder

RUN adduser -D -g 'bytebot' bytebot
WORKDIR /app
COPY . .
RUN apk add --no-cache git tzdata
RUN ./docker-build.sh

FROM scratch
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/group /etc/group
COPY --from=builder /app/opt/bytebot /opt/bytebot

USER bytebot
ENTRYPOINT ["/opt/bytebot"]
```

And add our services, namely the gateway and the redis db it's talking to:

```yaml
version: "3.8"
services:
  bytebot:
          #network_mode: "host" #TODO allows access to localhost
    build: .
    environment:
      BYTEBOT_REDIS: "redis:6379"
      BYTEBOT_INBOUND: ${BYTEBOT_INBOUND:-irc-inbound}
      BYTEBOT_OUTBOUND: ${BYTEBOT_OUTBOUND:-irc}
      BYTEBOT_FEED: "http://127.0.0.1:8000/flux.xml"
  redis:
    image: redis:6.2.3
    ports:
      - "127.0.0.1:6379:6379"
```
