# 03. Streams: Puddle to Puddle

A **stream** is the connection between two Puddles: private, peer to peer, both ways signed. The routes are
`/stream/v1/...`; the full list with bodies is Puddle's `api/federation.yaml`.

## Becoming connected

1. **A request to an address** (`POST /stream/v1/access`): someone knows only an address (from a `puddle://` link). The
   request carries the asker's chain and a one-time ticket, and is signed for recipient `*`. The answer is identical
   whatever the recipient decides (unsigned, part 02), so nobody learns who lives there or whether they said no.
2. **The answer** (`POST /stream/v1/access/result`), sent later from the recipient to the asker, carries the
   recipient's chain and the ticket. Only now does the asker learn who answered, by keys.
3. From then on every request on the stream is signed and checked against the chain held (part 02).

## While connected

- **Events** (`POST /stream/v1/events`): messages and group events, each **signed separately by its author** over its
  exact bytes and naming the key (`kid`), so history signed before a key roll still verifies. Encrypted end to end.
- **Keys** (`POST /stream/v1/identity`): a new chain after a roll, proven by the chain.
- **Address** (`POST /stream/v1/address`): where to reach the sender now, as a signed **address record**:

      {"record":"{\"type\":\"address\",\"id\":..,\"addresses\":[..],\"seq\":N,\"ts\":..}","sig":"<kid>:<base64>"}

  `sig` is over `puddle-address-v1\n` followed by the record's exact bytes, by the sender's current key. The highest
  `seq` wins. Taking one sends whatever was waiting for the sender at once. (A `tls_fp` from older Puddles is ignored.)
  Behind a Levee the record also names the Levee (part 06).
- **Profiles, pictures, files** (`/stream/v1/profile`, `/stream/v1/media`, `/stream/v1/files/...`).
- **Closing** (`POST /stream/v1/close`).

## Proving who answers

Every route here but the four that set a stream up (`access`, `access/result`, `access/status`, `identity`) needs the
handshake first (`GET /stream/v1/hello`, part 02). A caller holding the peer's key treats any answer without a good
receipt as **not delivered**, keeps it, and tries again on its own schedule (Puddle: every five minutes for a day, then
every thirty), telling its owner why ("didn't recognise us", "couldn't prove it's them", "intercepted").
