## Bytebot

Bytebot is:

 - actually several microservices in a trenchcoat pretending to be one bot
 - really just a lightweight message gateway and router with some lambdas attached
 - an experiment in message routing

Bytebot is not:
 - a chatbot (on its own)
 - a bot that does anything useful
 - supported

## How It Works

This system has three components: a message router, a message gateway, and a message consumer. 

```
                     ------------------
					 / Discord server /
					 ------------------
							  |
				 Inbound messages or events
							  |
							  v
					-------------------
					/ Message gateway /
					-------------------
							  |
		  Serialize message to JSON and send to message router
		  			(Write it to Redis)
							  |
		    Publishes messages to a hierarchical topic 
		 inbound.discord.{server id}.{channel id}.{user id}
							  |
							  v
					 ------------------
					 / Message router / 					
					 ------------------
							  |
			 Broadcast message to all subscribers
							  |
							  v
	-------------------         	------------------
	/ Subscriber foo /				/ Subscriber bar / 
	-------------------				------------------
							  |
				Decide what to do with message
							  |
		Respond by sending a message to an outbound topic
	   outbound.discord.{server id}.{channel id}.{user id}
	   						  |
							  v
					-------------------
					/ Message router /
					-------------------
							  |
		Serialize message to JSON and send to message gateway
							  |
							  v
					-------------------
					/ Message gateway /
					-------------------
							  |
				Subscribes to outbound topic pattern
						outbound.discord.*
							  |
		  Uses topic pattern to decide where to send message
							  |
							  v
					 ------------------
					 / Discord server /
					 ------------------
```

The message router is a Redis pub/sub server that receives messages from the gateway and publishes them to a topic on the message router (Redis pub/sub in this case). 

The message gateway is a bot that listens for messages from a specific protocol (e.g. Discord) and forwards them to the message router. The message consumer is a bot that listens for messages from the message router and does something with them (e.g. post them to Slack).