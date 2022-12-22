# nostr-relay-ln

This is a [nostr](https://github.com/nostr-protocol/nostr) relay, written in Rust. It currently supports the entire relay protocol, and persists data with SQLite.

This is a fork of [https://github.com/scsibug/nostr-rs-relay](https://github.com/scsibug/nostr-rs-relay).

## Features

[NIPs](https://github.com/nostr-protocol/nips) with a relay-specific implementation are listed here.

- [x] NIP-01: [Basic protocol flow description](https://github.com/nostr-protocol/nips/blob/master/01.md)
  * Core event model
  * Hide old metadata events
  * Id/Author prefix search
- [x] NIP-02: [Contact List and Petnames](https://github.com/nostr-protocol/nips/blob/master/02.md)
- [ ] NIP-03: [OpenTimestamps Attestations for Events](https://github.com/nostr-protocol/nips/blob/master/03.md)
- [x] NIP-05: [Mapping Nostr keys to DNS-based internet identifiers](https://github.com/nostr-protocol/nips/blob/master/05.md)
- [x] NIP-09: [Event Deletion](https://github.com/nostr-protocol/nips/blob/master/09.md)
- [x] NIP-11: [Relay Information Document](https://github.com/nostr-protocol/nips/blob/master/11.md)
- [x] NIP-12: [Generic Tag Queries](https://github.com/nostr-protocol/nips/blob/master/12.md)
- [x] NIP-15: [End of Stored Events Notice](https://github.com/nostr-protocol/nips/blob/master/15.md)
- [x] NIP-16: [Event Treatment](https://github.com/nostr-protocol/nips/blob/master/16.md)
- [x] NIP-20: [Command Results](https://github.com/nostr-protocol/nips/blob/master/20.md)
- [x] NIP-22: [Event `created_at` limits](https://github.com/nostr-protocol/nips/blob/master/22.md) (_future-dated events only_)
- [x] NIP-26: [Event Delegation](https://github.com/nostr-protocol/nips/blob/master/26.md)

## Hardware / Virtual Machine

Using a virtual machine allows you to scale your relay as the amount of data and users increases with time. For a private-ish, friends/family relay, you can start with an x64 based architecture, 2 GB RAM, 30 GB SSD. YMMV. Make sure you choose Ubuntu Server 20.04 or 22.04. Generate a SSH keypair file and save it locally. Ensure that the server is available via SSH (22), HTTP (80), and HTTPS (443).

## Installation

The sample [`config.toml`](config.toml) file demonstrates the
configuration available to the relay.  This file is optional, but may
be mounted into a docker container like so:

```console
$ docker run -it -p 7000:8080 \
  --mount src=$(pwd)/config.toml,target=/usr/src/app/config.toml,type=bind \
  --mount src=$(pwd)/data,target=/usr/src/app/db,type=bind \
  nostr-rs-relay
```

Options include rate-limiting, event size limits, and network address
settings.

## Reverse Proxy Configuration

For examples of putting the relay behind a reverse proxy (for TLS
termination, load balancing, and other features), see [Reverse
Proxy](reverse-proxy.md).

## Dev Channel

For development discussions, please feel free to use the [sourcehut
mailing list](https://lists.sr.ht/~gheartsfield/nostr-rs-relay-devel).
Or, drop by the [Nostr Telegram Channel](https://t.me/nostr_protocol).

To chat about `nostr-rs-relay` on `nostr` itself; visit our channel on [anigma](https://anigma.io/) or another client that supports [NIP-28](https://github.com/nostr-protocol/nips/blob/master/28.md) chats:
 * `2ad246a094fee48c6e455dd13d759d5f41b5a233120f5719d81ebc1935075194`

License
---
This project is MIT licensed.
