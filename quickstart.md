# Quickstart

### Overview

Simply put, Bytebot is a framework for managing a pub/sub architecture. Messages come in from outside soures, are written to the queue, and then dispatched to subscribers. In our quickstart, we will create a Discord bot, connect a gateway, then connect an app. For this example you will need the following:

- [Docker]()
- [Docker-compose]()
- The ability to pull images from a custom registry
- A discord bot token. You can get one by going [here](https://discordapp.com/developers/applications/), creating a new application, then selecting "bot" -> "Add bot" to convert your Discord application to a bot.

### Add your new bot to a server
From the left-hand panel of the discord developer portal, select your new bot app, then select OAuth2. Under scopes, click "bot", and under bot permissions, select the things you would like your bot to do. In general, you will want to have permission to see channels, send messages, and receive message history.

Once you have made your selections, Discord will generate a URL for you to use to invite the bot to your server. Copy that, paste it into a browser, and your bot will now join your server.

### Start the gateway and router
Bytebot relies on redis for message routing and caching some information, so we will launch a single-node redis deployment along with our gateway

Create a new directory and in that directory, add the following file as `docker-compose.yaml`

```
version: "3.8"
services:
  bytebot:
    image: dkr.fraq.dev/bytebot/gateway-discord:edge
    environment:
      BYTEBOT_REDIS: "redis:6379"
      BYTEBOT_ID: ${BYTEBOT_ID:-discord}
      BYTEBOT_INBOUND: ${BYTEBOT_INBOUND:-discord-inbound}
      BYTEBOT_OUTBOUND: ${BYTEBOT_OUTBOUND:-discord-outbound}
      BYTEBOT_TOKEN: your-discord-bot-token-here
  redis:
    image: redis:6.2.3
    ports:
      # So we can connect and watch messages on the queue if we need to
      - "127.0.0.1:6379:6379"
```

Next, run `docker-compose up -d` from this new directory and `docker-compose ps` to see the status of your containers. If all went as expected, you should have two containers running.

### Verify your deployment
To see your gateway in action, run `docker-compose logs -f bytebot` to see your bot connecting to discord. Every message sent is relayed through the discord gateway to redis, ready for apps to receive.
