# 04. Devices: a phone and its Puddle

A phone client is a **thin client** of its owner's Puddle: no identity keys, no chats. It has a key of its own (part 01,
device ids), signs every request with it (part 02, with its device id as the sender and the Puddle's user id as the
recipient), and believes only answers carrying the Puddle's receipt. There is no token to steal.

## Pairing

1. The owner asks their Puddle for a link (`POST /api/v1/devices/pair`, owner token only):

       puddle-pair://<address>?id=<the Puddle's user id>&secret=<32 hex: 128 random bits>

   The secret is good for one use within ten minutes; asking again replaces it.
2. The phone makes its key and sends `POST /device/v1/pair` with `{"name", "key": <64 hex>, "secret"}`, **signed by
   that key**. A wrong, spent or stale secret is the usual `404`.
3. The answer (`202`) carries `device`, `check`, and `puddle: {id, chain}`, with a receipt. The phone checks the chain
   belongs to the id in the link, then the receipt against the chain's current key: from here it knows its Puddle by
   keys. It is **pending**.
4. The owner sees the name and the six-digit check code (part 01), the phone shows the same code, and the owner accepts
   (`POST /api/v1/devices/{device}/accept`) or declines.
5. The phone learns which from `GET /device/v1/status` (any device on the list), which is also its **handshake**: it
   carries nothing, and its answer brings the Puddle's chain, so a phone learns a key roll there.

## After pairing

An accepted device's signature is accepted **wherever the owner token is**, except managing devices (pairing links,
accepting, declining, revoking), which stays with the owner token so a stolen phone can't let anything else in. Every
answer to a device carries a receipt. Revoking a device (`POST /api/v1/devices/{device}/revoke`) makes its next
request a `404`; nothing about the identity changes.

Ponds come for free: the app reaches Ponds only through its own Puddle.
