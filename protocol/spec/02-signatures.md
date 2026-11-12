# 02. Signatures: who sent it, and who answered

Two headers, one in each direction. Together they mean a request is from who it says, meant for who it names, used
once, and that its answer came from who the sender meant, over the connection the sender saw.

## Request signatures

    X-Puddle-Signature: v1 id=<sender id>; kid=<key id>; ts=<ms>; nonce=<base64>; sig=<base64>

`sig` is Ed25519 by the key `kid` names, over these lines joined by `\n` (no newline at the end):

    puddle-request-v1
    <METHOD>
    <request target: path and query, exactly as sent>
    <hex SHA-256 of the body; of nothing for an empty one>
    <ts>
    <nonce>
    <the recipient's id>

The header is parsed strictly: exactly these fields, in this order, nothing extra. The nonce is 16 random bytes.

The recipient checks, and answers **`404` to any failure up to and including the signature**:

1. the recipient id is its own. Only a route meant for a first contact (a stream request to an address, joining a
   Pond) also accepts `*`.
2. the sender is one it knows and `kid` is their **current** key; or the route proves the key another way (a chain in
   the body, a device pairing). Retired keys can't start new requests.
3. the signature verifies.

Only then may it say what is wrong: `401 clock_skew` (with its `now`) when `ts` is more than five minutes off,
`401 replayed` when the nonce was seen in that window, and the route's own refusals.

## Answer receipts

Every answer to a request that authenticated, from someone the responder already holds a record of, carries:

    X-Puddle-Receipt: v1 id=<responder id>; kid=<key id>; fp=<hex>; sig=<base64>

`fp` is the hex SHA-256 of **the TLS certificate on this connection, as the responder sees its own side** (normally its
own certificate; behind a Levee, the Levee's: part 06). `sig` is Ed25519 by the responder's current key over:

    puddle-receipt-v1
    <the responder's id>
    <the request's nonce>
    <the HTTP status>
    <hex SHA-256 of the answer's body; of nothing for an empty one>
    <fp>

**No receipt** on: the plain `404` a stranger gets; any answer to a stranger, even one who proved a request with a
chain of their own; and the routes that must answer everyone alike (a stream request and its status), so that a
receipt never tells anyone who lives at an address or whether they are blocked.

An answer sent straight from a file (any part of it, with `Range`) is signed over exactly the bytes sent.

### Checking a receipt (MUST, for any caller that holds the responder's key)

Accept an answer only if the receipt is there, names the id and key the caller holds for the responder, carries this
request's nonce, the status and body received, and **the fingerprint of the certificate the caller saw on this
connection**. That last check catches someone in the middle: they can pass a request to the real party and its signed
answer back, but the real party signs its own side of a different connection.

What a caller makes of the result:

| result | meaning |
|---|---|
| good receipt, `2xx` | delivered |
| good receipt, any other status | refused, in their name: final |
| no receipt (a plain `404` and the like) | not them, or they don't know us: **not delivered**, try again later |
| another key, or a signature that doesn't check | not proven: **not delivered** |
| good signature, other certificate | intercepted: **not delivered** |

### The handshake

A receipt arrives after the request has gone. So before sending anything that needs an existing relationship, a caller
that holds the responder's key sends a request that carries nothing (`GET /stream/v1/hello` between Puddles,
`GET /device/v1/status` from a phone) and checks its receipt. That proves which certificate is the responder's; the real
request then goes only over a connection showing that certificate. A proof may be kept for ten minutes, each good
receipt renewing it.

Vectors: `vectors/signatures.json` (signed requests and receipts, with what a checker must say about each).
