# 05. Ponds: a Puddle and a community server

A **Pond** is a community server, like a Discord server: channels, many members, history kept **on the Pond**. A
member's Puddle caches what it has seen. There is no peer to peer here: members talk to the Pond, never to each other
through it, and the Pond never calls a Puddle. The routes are `/pond/v1/...`; Pond's `api/pond.yaml` and
`docs/design.md` have them in full.

## Joining and being a member

- **Invite links**: `pond://<address>?invite=<26 base32>`. A used-up, expired, revoked or unknown invite is `404`.
- **Joining** (`POST /pond/v1/join`): `{invite, chain, profile}`, signed for recipient `*` (the Puddle doesn't know the
  Pond's id yet), proven by the chain in the body. The answer carries the Pond's id, name, channels and event cursor.
- **Every other route** is signed for the Pond's id by the member's current key (part 02). A non-member gets `404`.
- **Keys**: a member that rolls its keys sends its new chain to `POST /pond/v1/keys`.
- **Events**: one request held open on `GET /pond/v1/events?after=<cursor>` (up to 25 seconds). Because the Puddle
  always makes the connection, this works from behind NAT and through a Levee.
- **What a Pond can do** is in `/pond/v1/info`: `protocol` (a number) and `features` (`files`, ...).

## Trust today, and the change proposed here

Today a Puddle trusts a Pond the way it once trusted Puddles: it **pins the Pond's TLS certificate** at join and refuses
any other (`certificate_changed`). A renewed certificate (Let's Encrypt every 90 days, say) cuts every member off until
they join again.

**Proposed (protocol 2 of `/pond/v1`, feature `receipts`):** a Pond signs a receipt (part 02) on every answer to an
authenticated member, with its Pond key, over its own certificate. A Puddle that sees `receipts` in `features`:

- checks the receipt on every answer (the Pond's id and key are those from join), and
- stops pinning: a new certificate is simply proven again by the next receipt.

The Pond's `GET /pond/v1/info` doubles as the handshake. A Pond without the feature is pinned as today. This needs no
new route, only the header Pond already has the pieces for (its key, its certificate).

## Later (protocol, not built)

- **Messages signed by their authors**, so a Pond can't forge what a member said (members trust the Pond for what it
  shows today, as they trust a Discord server). The event format would be a Puddle event (part 03): the author's
  signature over exact bytes, naming `kid`.
- **Roles and moderation** travel as Pond events like any other.
