# goChan: A little package that can stream events to the web

[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Frajveermalviya%2Fgochan.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Frajveermalviya%2Fgochan?ref=badge_shield)
[![GoDoc](https://godoc.org/github.com/rajveermalviya/gochan?status.svg)](https://godoc.org/github.com/rajveermalviya/gochan)
[![Go Report Card](https://goreportcard.com/badge/github.com/rajveermalviya/gochan)](https://goreportcard.com/report/rajveermalviya/gochan)

⚠ This project is on early stage, it's not ready for production yet ⚠

goChan is a small package for relaying pubsub messages to the web via
[SSE(Eventsource)](https://en.wikipedia.org/wiki/Server-sent_events).
It is based on Twitter's implementation for real-time event-streaming
in their new web app.

It uses the [Go CDK](https://gocloud.dev) as vendor neutral pubsub driver that supports multiple pubusub vendors:

- Google Cloud Pub/Sub
- Amazon Simple Queueing Service (Pending)
- Azure Service Bus (Pending)
- RabbitMQ
- NATS
- Kafka
- In-Memory (Only for testing)

## Installation

goChan supports Go modules and built against go version 1.13

```shell
go get github.com/rajveermalviya/gochan
```

## Documentation

For documentation check [godoc](https://godoc.org/github.com/rajveermalviya/gochan).

## Usage

goChan uses Server-Sent-Events, because of this it doesn't require to run a standalone server, unlike websockets it can be embedded in your api server.
goChan's streamer has a ServeHTTP method i.e it implements http.Handler interface so that it can be used directly or can be wrapped with middlewares like Authentication easily.

```go
// Using streamer directly
streamer, err := gochan.NewStreamer(ctx, &gochan.ConfigStreamer{
	Driver:       drivers.DriverMem,
	DriverConfig: &drivers.ConfigMem{},
})

log.Fatal("HTTP server error: ", http.ListenAndServe("localhost:3000", streamer))
```

```go
// Using streamer by wrapping it in auth middleware
streamer, err := gochan.NewStreamer(ctx, &gochan.ConfigStreamer{
	Driver:       drivers.DriverMem,
	DriverConfig: &drivers.ConfigMem{},
})

mux := http.NewServeMux()
mux.HandleFunc("/events", func (w http.ResponseWriter, r *http.Request) {
    err := Auth(w,r)
    if err != nil {
        return
    }
    streamer.ServeHTTP(w,r)
})
log.Fatal("HTTP server error: ", http.ListenAndServe("localhost:3000", mux))
```

When client connects to the server it will send two system messages upfront 1. Configuration: it contains the client-id and client-timeout and 2. Subscriptions associated with the specified client id. These messages have topic prefixed with '/system'
New client is created explicitly using the 'streamer.NewClient' method for client with auto generated id. Or 'streamer.NewCustomClient' method for client with specified id.

To generate a new client-id everytime a client connects use the following middleware with the streamer

```go
mux.HandleFunc("/events", func(w http.ResponseWriter, r *http.Request) {
    // Auto generate new client id, when new client connects.
    q := r.URL.Query()
    if q.Get("id") == "" {
        client, _ := streamer.NewClient(ctx)
        q.Set("id", client.ID)
        r.URL.RawQuery = q.Encode()
    }

    streamer.ServeHTTP(w, r)
})
```

When a client gets disconnected it has a time window to connect to the server again with the state unchanged.

Check out the [example](examples/1000topics)

## Why Server Sent Events (SSE) ?

Why would you choose SSE over WebSockets?

One reason SSEs have been kept in the shadow is because later APIs like WebSockets provide a richer protocol to perform bi-directional, full-duplex communication. However, in some scenarios data doesn't need to be sent from the client. You simply need updates from some server action. A few examples would be friends' status updates, stock tickers, news feeds, or other automated data push mechanisms (e.g. updating a client-side Web SQL Database or IndexedDB object store). If you'll need to send data to a server, Fetch API is always a friend.

SSEs are sent over traditional HTTP. That means they do not require a special protocol or server implementation to get working. WebSockets on the other hand, require full-duplex connections and new Web Socket servers to handle the protocol. In addition, Server-Sent Events have a variety of features that WebSockets lack by design such as automatic reconnection, event IDs, and the ability to send arbitrary events.

Because SSE works on top of HTTP it always stays updated. For example HTTP/3 will use QUIC protocol as transport layer that improves data streaming and many more improvements.

## Future Goals:

- Support additional pusub vendors by contributing back to [Go CDK](https://gocloud.dev)
- Standalone server that can be configured by yaml, while also staying modular.
- Become a [CNCF](https://cncf.io) project (...maybe)

## Users

If you are using gochan in production please let me know by sending an [email](mailto:rajveer0malviya@gmail.com) or file an issue

## Show some love

The best way to show some love towards the project, is to contribute and file issues.

If you **love** gochan, you can support by sharing the project on Twitter.

You can also support by sponsoring the project via [PayPal](https://paypal.me/rajveermalviya).

## License

[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Frajveermalviya%2Fgochan.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Frajveermalviya%2Fgochan?ref=badge_large)
