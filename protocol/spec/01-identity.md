# 01. Identity

## A Puddle's identity

Three keys, all derived from one recovery phrase (shown to the owner once, never stored):

- an **Ed25519 signing key**: signs requests, receipts, records and events
- an **X25519 encryption key**: end-to-end encryption between Puddles
- an Ed25519 **roll key**: only authorizes changing the keys; exists only while the phrase is typed in

**User id** = `base32(SHA-256(first signing public key || SHA-256(first roll public key)))`. It never changes when keys
roll. Covering the roll key's hash means someone who steals only the daily keys can't make a chain for the same id.

**Key id** (`kid`) = hex SHA-256 of a signing public key. Requests and signed events name the key that signed them.

## The key chain

What others are given to check an identity (`chain` in requests, `identity/chain.jsonl` on disk): one JSON record per
line, each ending in `\n`. A checker MUST take each `record` value's exact bytes from the line (from its `{` to its
`}`): signatures cover those bytes, never a re-serialisation.

**Inception** (first line):

    {"record":{"type":"identity.inception","seq":0,"signing":"<b64>","encryption":"<b64>","next":"<hex>","ts":<ms>},"sig":"<b64>"}

`next` is the SHA-256 of the first roll key. `sig` is by `signing`. The id is `base32(SHA-256(signing || next's 32 bytes))`.

**Rotation** (each later line):

    {"record":{"type":"identity.rotate","seq":N,"prev":"<hex>","signing":"<b64>","encryption":"<b64>","roll":"<b64>","next":"<hex>","ts":<ms>},"sig":"<b64>","sig_roll":"<b64>","sig_prev":"<b64>"}

`seq` is one more than the record before; `prev` is the SHA-256 of that record's bytes; `roll` reveals the key the
record before committed to as `next`. `sig` is by the new signing key, `sig_roll` by `roll`, `sig_prev` by the previous
signing key, all over this record's bytes.

A chain is valid when every line checks. The current keys are those of the last record. Fields are found by name; blank
lines are skipped; a chain is at most 4 MiB and 10,000 records.

A peer that rolls its keys pushes its new chain to everyone it talks to (`POST /stream/v1/identity`,
`POST /pond/v1/keys`), signed by the new key and proven by the chain itself; a receiver takes it only if it extends the
chain it already holds for that id.

## Pond ids

A Pond has one Ed25519 key and no chain: its id is `base32(SHA-256(public key))`. No phrase, no rolling; a Pond that
loses its key is a new Pond.

## Device ids

A phone has its own Ed25519 key, never the identity's: its id is `base32(SHA-256("puddle-device-v1\n" || public key))`,
the shape of a user id but never one. Its **check code**, shown on both screens at pairing, is the first four bytes of
that same hash as a big-endian number, modulo 1,000,000, as six digits with leading zeros. Part 04.

## Levee ids

A Levee has one Ed25519 key, like a Pond: its id is `base32(SHA-256("puddle-levee-v1\n" || public key))`. Part 06.

Vectors: `vectors/devices.json` (device ids and check codes from fixed seeds).
