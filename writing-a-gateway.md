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
It's a simple readonly gateway, so we won't use the inbound queue. FIXME?


### Deploying a gateway

TODO
