pinpriq
=====

networked priority queue

Goals
-----

A priority queue. Clients attach through TCP. Available operations are `push`,
`pop`, `subscribe`, and `unsubscribe`.

The available abstractions are the `queue` and the `packet`. A `queue` is a set
of `packet`. A `queue` is created on demand by pushing to it.

A `packet` consists of a `key` and a `payload`.

Each packet has a time to live (`TTL`). Once that time has passed the packet is
removed from the `queue`.

There are two data types; byte sequences and integers. On the wire, integers
are encoded as big endian unsigned.

The `packet` is a sequence of bytes laid out like this;

```
0       1       2       3       4
| key len       | payload len   | key ... | payload ...
```

Key length and payload length units are bytes.

`push` - push a packet to a queue. There can be more than one packet with the
same key. Packet metadata is a `TTL` (in milliseconds), and a `priority` (with
1 being the highest priority, and 0 being no priority).

The `push` is a byte sequence like this;
```
0       1       2       3       4       5       6
| 0x70  | queue name len| TTL           |  prio | packet... | queue name...
```

There is no return value.


`pop` - pop a `packet` from a `queue`. The `packet` will be the one with the highest
`priority` (`priority` 1 will be selected before priority 2). If there are
more than one such `packet`, it will be the newest one, (i.e. the last one
inserted). If there are other packets with the same key, they will also be
popped. A `packet` with priority 0 is not selectable, but will be returned as a
side effect if it has the same key as the selected `packet`.

The `pop` is a byte sequence like this;

```
0       1       2       3
| 0x50  | queue name len| queue name...
```

Return value is a byte sequence like this;

```
0       1       2
| number of pkts| pkt... | pkt...
```

If the queue is empty, the return value will be;

```
0       1       2
| 0x0           |
```

`subscribe` - subscribe to a queue. A `packet` will be sent as soon as one
becomes available. If there are more than one available `packet`, the highest
priority one will be selected as per `pop`. Redundant `subscribe` are ignored.
Note that a `packet` with priority 0 is never a candidate for selection; it can
only be delivered as a side effect of having the same key as a selected packet.

```
0       1       2       3
| 0x73  | queue name len| queue name...
```

There is no reply.

When subscribing, the client indicates that it is ready to receive a new
`packet` by sending the single byte `0x41`.

`unsubscribe` - unsubscribe from a queue. Redundant `unsubscribe` are ignored.

```
0       1       2       3
| 0x75  | queue name len| queue name...
```

There is no reply.


Build
-----

    $ rebar3 compile

