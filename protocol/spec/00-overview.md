# 00. Overview

## Who talks to whom

```
        phone client ──(devices)──▶ Puddle ◀──(streams)──▶ Puddle
                                     │  ▲
                            (ponds)  │  │ (levee: the Puddle connects out; others reach it here)
                                     ▼  │
                                    Pond  Levee
```

- **Puddle to Puddle** (`/stream/v1/...`): streams, the private, peer-to-peer part. Part 03.
- **Phone client to its Puddle** (`/api/v1/...` and `/device/v1/...`): the owner API, signed by a device key. Part 04.
- **Puddle to Pond** (`/pond/v1/...`): a member and a community server. The Puddle always calls; the Pond never calls
  a Puddle. Part 05.
- **Puddle and its Levee** (`/levee/v1/...`): the Puddle attaches, and anyone calling the Puddle reaches the Levee. Part 06.

Every party has an **Ed25519 identity**, and everything that matters is signed by it. Part 01.

## Principles (every part follows these)

1. **Keys, not certificates.** TLS is always used and only encrypts. Every party makes its own certificate, nobody checks
   it against a CA, and it may change at any time. Who a party is, is proven by its keys: requests carry the sender's
   signature, answers carry the responder's receipt (part 02), which also names the certificate the connection showed,
   so someone in the middle is caught. (A Puddle still pins a Pond's certificate until Ponds sign receipts: part 05.)
2. **Strangers learn nothing.** A request that fails any check up to and including its signature is answered `404`,
   exactly as a path that does not exist, never `401` or `403`. Only an authenticated sender is told what went wrong.
   Routes that must answer everyone the same (asking for a stream) never sign their answer.
3. **GET reads, POST does.** Only GET and POST. GET never changes state. POST carries the verb in the path
   (`.../accept`, `.../delete`). No PUT, PATCH, DELETE or HEAD.
4. **JSON over HTTPS, bodies limited per route**, decided before a body is read. Unknown fields are ignored.
5. **Ids are self-certifying.** A user id (or Pond id, or device id) is a hash of keys, so given the id and the key
   chain anyone can check they belong together, with no registry.

## Versioning

- The path says the protocol version: `/stream/v1`, `/pond/v1`, `/device/v1`, `/levee/v1`. A change an older
  implementation would misread gets new paths (`v2`), served beside the old ones until nobody uses them.
- Within a version, things are only **added**: routes, fields, event kinds, headers. An implementation ignores what it
  does not know, so older and newer keep working together.
- A party that has optional parts says so: a Pond's `/info` carries `protocol` (a number that goes up with each
  addition) and `features` (names: `files`, `receipts`, ...). A caller checks these before using an addition, rather
  than guessing from a `404`, which is also what a refusal looks like.
- Signed strings carry their own version label (`puddle-request-v1`, `puddle-receipt-v1`, `puddle-address-v1`), so a
  signature made for one purpose can never be taken for another.

## Encodings (everywhere)

| What | Encoding |
|---|---|
| user, Pond, device ids | RFC 4648 base32, lowercase, no padding: 52 characters |
| key ids (`kid`), hashes | 64 lowercase hex characters (SHA-256) |
| keys, signatures in JSON and headers | standard base64 with padding (32 and 64 bytes) |
| times | UTC milliseconds since 1970, as JSON integers |
