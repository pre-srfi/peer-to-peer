# SRFI nnn: peer-to-peer

by Amirouche Boubekki

# Status

Early Draft

# Abstract

This library describe the primitives for a peer-to-peer network that
can be used to implement networked applications that do not require a
central server to work. It offers the building blocks to create
decentralized applications.

# Issues

??? Optional section that may point out things to be resolved. This
will not appear in the final SRFI.

# Rationale

It is very schemey to operate toward a common goal in a distributed
fashion. This library brings the distributed social characterics of
Scheme to Scheme programs.

Many Scheme implementations provides network support. This library
will allow them to exercise those capabilities while allowing them to
create something bigger than what they could achieve alone. It offers
a unique opportunity for Scheme implementations to cooperate toward a
common goal while all will keep their defining characteristics.

It is also an opportunity to bring fresh air to the Internet. This
library offers new possibilities to create networked applications in a
decentralized way that could spur creativity.

## Survey of prior art

GitHub's version of Markdown can make tables. For example:

| System        | Procedure | Signature                 |
| ------------- |:---------:| ------------------------- |
| System A      | `jumble`  | _list_ _elem_             |
| System B      | `bungle`  | _elem_ _list_             |
| System C      | `frob`    | _list_ _elem_ _predicate_ |

# Specification

The specifications prescribe the use of Ed25519 signing when using
namespaces.

The network protocol is based on serialized scheme expressions
compatible with `write` prefixed with the size of the serialized
expression in network byte order (TODO: big or little?). Because the
network can not be trusted the procedure used to parse messages from
the network must be safe.

The maximum size of serialized scheme expression transmitted over the
network know as message is defined to be 1 megabyte.

The protocol identifies peers with bytevectors of 32 bytes and use XOR
metric to find nearest peers similar to what kademlia paper describe
and similar to the "mainline dht". The protocol doesn't dictate a
particular datastructure for the routing table. Peer identifiers are
refered as `uid` pronounced "weed".

A peer initiating a request must wait for a reply at most 5 seconds.

The protocol is not compatible with existing distributed hash tables.

The default replication factor is 5. The maximum replication factor
that must be respected globaly is 20. TODO: explain what it is.

This library defines the following disjoint type: `peer`

### Public procedures

#### Initialization

##### `(peer uid recv send replication) → peer`

Returns a peer object identified as `UID` bytevector. `UID` must be a
randomly chosen bytevector of 32 bytes. `RECV` is thunk that allows to
retrieve incoming payload as a scheme object. `SEND` takes ip string,
port number and scheme object payload as argument and allow to send to
the given address the given payload. It is an error if the payload
serialize to a bytevector bigger than 1Mb. `REPLICATION` must be at
least 5 and at most 20.

A peer must remember its own `UID` for future use.

##### `(peer-connect peer ip port) → boolean`

This will make sure that `PEER` knows about the peer at `IP` and
`PORT`. The peer must send a `iam` network procedure. Return `#t` on
success. Otherwise `#f`.

##### `(peer-bootstrap peer) → unspecified`

`PEER` will initiate a bootstrap algorithm that aims at filling the
routing table based on known peers with other peers in its surrounding
and peers from the rest of the 256 bits space. It must rely on `peers`
network procedure described in the next section to discover new peers.

#### Distributed-Hash-Table procedures

##### `(peer-set peer value) → bytevector`

Set `VALUE` bytevector in the network at the nearest peers to the
SHA-256 of `VALUE`. Returns the SHA-256 of `VALUE` as bytevector.

##### `(peer-ref peer key) → bytevector`

Returns the value associated with `KEY` bytevector of 32 bytes in the
network. The returned value must have `key` as sha256.

#### Bag procedures

##### `(peer-bag-publish peer key value) → unspecified`

Adds `VALUE` bytevector of 32 bytes to the bag in the network
associated with `KEY` bytevector of 32 bytes. Bag values can be freely
added by any peers.

##### `(peer-bag-popular peer key) → vector`

Return most popular values associated with `KEY` bytevector of 32
bytes.

##### `(peer-bag-popular-at peer uid key) → vector`

Return most popular values associated with `KEY` bytevector of 32
bytes at the peer known as `UID`.

##### `(peer-bag-recent peer key)`

Return most recent values associated with `KEY` bytevector of 32
bytes.

##### `(peer-bag-recent-at peer uid key)`

Return most recent values associated with `KEY` bytevector of 32 bytes
at the peer known as `UID`.

##### `(peer-bag-sample peer key)`

Return random values associated with `KEY` bytevector of 32 bytes.

##### `(peer-bag-sample-at peer uid key)`

Return random values associated with `KEY` bytevector of 32 bytes
at the peer known as `UID`.

#### Namespace procedures

##### `(peer-namespace-set peer private-key public-key key value)`

Set in the network `VALUE` bytevector of 32 bytes associated with
`PUBLIC-KEY` and `KEY` bytevector of 32 bytes.  `PRIVATE-KEY` must be
used to compute the signature of `KEY` and `VALUE` as described later
in `namespace-set` network procedure.

##### `(peer-namespace-ref peer public-key key signature)`

Retrieve from the network the value associated with `PUBLIC-KEY` and
`KEY`. `KEY` and the returned value must be validated using
`PUBLIC-KEY` and `SIGNATURE`.

##### `(peer-namespace-ref-at peer uid public-key key signature)`

TODO

### Network procedures

The following procedures are encoded as Scheme vectors. The procedure
name is encoded as symbol inside the vector. Both request and replies
must not exceed 1 megabyte when encoded. If a message payload exceed 1
megabyte, the connection must be closed.

#### `iam uid`

A peer receiving this message must reply with a vector with the
following values:

- `welcome` symbol
- IP as a string (TBD)
- port as an integer
- `uid` as an integer

This allows peers to discover their external IPs. The receiving peer
may or may not add the initiating peer to its routing table and
initiate a `iam` network procedure.

#### `peers key`

`KEY` must be a 256 bits number. A peer receiving this message must
reply with a vector starting with `peer` followed by at most 20 (ip,
port) pairs that are the peers nearest to `KEY` known to the receiving
peer.

#### `set value`

The peer receiving this message must first check that it is part of
the nearest peers to the associated sha256 key of `VALUE` bytevector
based on the global replication factor. If the receiving peer knows
about peers that are more near the key, it must reply `#f` otherwise
it must store the `VALUE` bytevector and reply `#t`.

#### `ref key`

The peer receiving this message has two options for the reply:

- If it has the value associated with `KEY` it can return a vector
with two values the first being `value` symbol and the second the
bytevector associated with `KEY`.

- If it doesn't know about `KEY` it must return nearest known peers as
described in the above `peers` network procedure.

#### `bag-add key value`

The peer receiving this message must associate `KEY` to `VALUE` except
if it knows about peers that are more near `KEY` than itself and the
receiving peer is not part of the nearest peers. Similarly to `set`.

#### `namespace-set public-key key value signature`

TODO

#### `namespace-ref public-key key`

TODO

# Implementation

??? explanation of how it meets the sample implementation requirement
(see process), and the code, if possible, or a link to it Source for
the sample implementation.

# Acknowledgements

??? Give credits where credits is due.

# References

??? Optional section with links to web pages, books and papers that
helped design the SRFI.

# Copyright

Copyright (C) Amirouche BOUBEKKI (2020).

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
