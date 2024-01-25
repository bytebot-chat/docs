## Bytebot

Bytebot is:

 - actually several microservices in a trenchcoat pretending to be one bot
 - really just a lightweight message gateway and router with some lambdas attached
 - an experiment in message routing

Bytebot is not:
 - a chatbot (on its own)
 - a bot that does anything useful (yet)
 - supported (ever)

## How It Works

This system has three components: a message router, a message gateway, and a message consumer. 

Below is an example using two subscribers listening for messages from Discord.

![Flow of messages through bytebot's pubsub system][messageFlow]

[messageFlow]: https://mermaid.ink/img/pako:eNqNks1uwjAMx1_Fijgh4AE6aRLQgThsmsZuDYe08WhEm1T52IYo7z6XUMY2aaLpIXHs_8-xfWCFkcgStrWiKeE1veOa-2mWKlcYK8GhfUe7gfH4HtqVzk3QEmp0TmzRwZs1NZxdW5hlj_ECtsLjh9hvTmKzGAzPIa-UK_to8Ib-RhUJ5xpUlJ7IKDYZRPAqnQyKUmiNVbcNrjO1MM9ekDw7SRfyDgP0zc-gNVkLqxqvjIZGeI9WA34WVZCU84lJyCfje88cJYU9ZK4_2uRvQsaXaNd9VsOboEpfQ7uHXhEpKP0XeWGkvyposan2YDTpmeBvLxxJLM6Va2Llov7i5xu65Kg5f6SH0C4vLV6eWxwVlr0Cfo8H5KLYdUIxI2inNA202IjVaGuhJI3doYvnjGpbI2cJbaWwO864PpKfCN6s97pgibcBRyw0krCpEjStdTQevwA-9PUM?type=png