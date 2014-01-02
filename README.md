# Rmux #

Rmux is a Redis connection pooler and multiplexer, written in Go.  

## Installing ##

- Install [Go](http://golang.org/doc/install) 
- go get -u github.com/Pardot/rmux
- go build -o /usr/local/bin/rmux github.com/Pardot/rmux/main


## Usage ##

```
Usage of rmux:
  -host="localhost": The host to listen for incoming connections on
  -localReadTimeout=0: Timeout to set locally (read)
  -localTimeout=0: Timeout to set locally (read+write)
  -localWriteTimeout=0: Timeout to set locally (write)
  -maxProcesses=0: The number of processes to use.  If this is not defined, go's default is used.
  -poolSize=50: The size of the connection pools to use
  -port="6379": The port to listen for incoming connections on
  -remoteConnectTimeout=0: Timeout to set for remote redises (connect)
  -remoteReadTimeout=0: Timeout to set for remote redises (read)
  -remoteTimeout=0: Timeout to set for remote redises (connect+read+write)
  -remoteWriteTimeout=0: Timeout to set for remote redises (write)
  -socket="": The socket to listen for incoming connections on.  If this is provided, host and port are ignored
  -tcpConnections="localhost:6380 localhost:6381": TCP connections (destination redis servers) to multiplex over
  -unixConnections="": Unix connections (destination redis servers) to multiplex over
```

Localhost example:
```
redis-server --port 6379 &
redis-server --port 6380 &
redis-server --port 6381 &
redis-server --port 6382 &
rmux -socket=/tmp/rmux.sock -tcpConnections="localhost:6379 localhost:6380 localhost:6381 localhost:6382" &
redis-cli -s /tmp/rmux.sock
```

- In the above example, all key-based commands will hash over ports 6379->6382 on localhost
- All servers running production code should be running the same version (and destination flags) of rmux, and should be connecting over the rmux socket
- Select will always return +OK, even if the server id is invalid
- Ping will always return +PONG
- Quit will always return +OK
- Info will return an abbreviated response:

```
rmux_version: 1.0
go_version: go1.1.2
process_id: 48885
connected_clients: 0
active_endpoints: 4
total_endpoints: 4
role: master
```

Production equivalent:
```
rmux -socket=/tmp/rmux.sock -tcpConnections="redis1:6379 redis1:6380 redis2:6379 redis2:6380"
```

### Disabled commands ###

Redis commands that should only be run directly on a redis server are disabled.  Commands that operate on more than one key (or have the potential to) are disabled if multiplexing is enabled.

PubSub support is currently experimental, and only publish and subscribe are supported.
Disabled:
```
psubscribe
pubsub
punsubscribe
unsubscribe
```

[Full list of disabled commands](DISABLED_COMMANDS.md)

### Benchmarks ###

Benchmarks with keep-alive off (simulating a lamp stack) show rmux being ~4.5x as fast as a direct connection, under heavy load:

Benchmarks with keep-alive on (simulating how a java server would operate) show a direct connection to a redis server server being ~2.2x as fast:

[Benchmark results here](BENCHMARKS.md)