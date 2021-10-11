# go-push

An extensible general message push prototype implemented by golang.

# Install

Upgraded to golang 1.13, based on gomod management dependency.

* download go-push

```
go get github.com/owenliang/go-push
```

* Installation dependent environment

```
export GOPROXY=goproxy.io
go mod download
```

* Compiling Gateway Services

```
cd gateway/cli && go build && cd -
```

* Compiling logic services

```
cd logic/cli && go build && cd -
```

# Structure

* Gateway: long connection gateway
    * Massive long connections are broken up according to the bucket to reduce the lock granularity of push traversal
    * Message pre merging according to broadcast / room granularity reduces coding CPU loss, reduces system network calls, and greatly improves throughput
* Logic: logical server
    * It is stateless and responsible for distributing push messages to all gateway nodes
    * Expose the HTTP / 1 interface to the caller to facilitate business docking
    * Use HTTP / 2 long connection RPC to distribute messages to the gateway cluster

# Potential problems

* The main bottleneck of push is the gateway layer rather than internal communication, so small packet communication is still used between gateway and logic (PPS pressure on network card). At the same time, logic provides batch push interface for services to alleviate special requirements

# Benchmark

## environment

* 16 vcore
* client, logic, gateway deployed together


# Push API for logic

* Full broadcast

```
curl http://localhost:7799/push/all -d 'items=[{"msg": "hi"},{"msg": "bye"}]'
```

* Room broadcast

```
curl http://localhost:7799/push/room -d 'room=default&items=[{"msg": "hi"},{"msg": "bye"}]'
```

## gateway的websocket协议

* PING(client->server)

```
{"type": "PING", "data": {}}
```

* PONG(server->client)

```
{"type": "PONG", "data": {}}
```

* JOIN(client->server)

```
{"type": "JOIN", "data": {"room": "fengtimo"}}
```

* LEAVE(client->server)

```
{"type": "LEAVE", "data": {"room": "fengtimo"}}
```

* PUSH(server->client)

```
{"type": "PUSH", "data": {"items": [{"name": "go-push"}, {"age": "1"}]}}
```