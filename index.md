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


<div class="mermaid">
graph TD;
	A[Discord server] --> B[Message gateway];
	B --> C[Message router];
	C --> D[Message consumer foo];
	C --> E[Message consumer bar];
	D --> F[Message router];
	E --> F;
	F --> G[Message gateway];
	G --> H[Discord server];
</div>