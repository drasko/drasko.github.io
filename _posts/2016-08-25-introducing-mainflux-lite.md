---
layout: post
published: true
mathjax: false
featured: false
comments: true
---
## Taming the Mainflux Development Hurdle

So, you tested [Mainflux](https://github.com/Mainflux/mainflux) and you like it. You want to get your feet wet in new technologies - like [Go](https://golang.org/), [NATS](http://nats.io/) or [InfluxDB](https://influxdata.com/). Or you are familiar with these already and you want to modify and customize Mainflux to your needs.

It's easy - you did `git clone` of all the microservices repos and... what now?

Weel - you must surely start NATS broker. And then mainflux Core. Which will connect to MongoDB (so you better start Mongo first). And finally you will have to start some of the API servers - for example [mainflux-http-server](https://github.com/Mainflux/mainflux-http-server).

OK, so far so good.

Now you will need an additional terminal to shoot `curl` requests from:

```bash
curl -s -i -H "Accept: application/json" -H "Content-Type: application/json" localhost:7070/status | json | pygmentize -l json
```
This should work but the problem is obvious - changing one think will demand recompilation of several microservices that are affected, and keeping all this set-up in sync is always fun. It is not too difficult and yes - things can be automatized, but well... it's still time consuming.

Here is where [Mainflux Lite](https://github.com/Mainflux/mainflux-lite) comes to rescue.

Mainflux Lite is actually several Mainflux microservices bundled into a single monilite binary. We took API server and DB backend and bundled them into one. We trow out NATS broker for now (not needed in this combo) and the only external thing that is still needed is DB (MongoDB for now).

As a result we have one Go program that compiles quickly and is easy to change - bot API side and DB backend. This way we can develop quickly.

It is also simpler to deploy the server - so when we prototype sensor setup or we do not need full-blown  Mainflux system then Mainflux Lite can be perfect (for example embedded in RPi for IoT gateway purposes).

Good side is that any change can be very easily ported to microservices, as they are 1:1 mapped, so this  falls back to literal copy-paste of the code.

Mainflux Lite has thus many purposes and one of the main is lowering the entry barrier. If you are new to Mainflux systema and enthisiastic to learn about this moder technology - them mainflux Lite is a great place to start.
