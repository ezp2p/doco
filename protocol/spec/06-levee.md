# 06. Levee: a stable front for a Puddle

A **Levee** is a relay a Puddle sits behind, run by someone competent on a server with a public address (rented, or a
VPS of one's own). The Puddle connects **out** to it and needs no open ports. Friends add the Puddle by the **Levee's
address**, which stays the same when the Puddle's home network changes. Everything sent to the Puddle arrives at the
Levee: passed straight through while the Puddle is attached, **spooled** while it is not, and delivered when it comes
back. What the Puddle sends can go out through the Levee too, and is spooled there if the far end is away. **Chats stay
on the Puddle**: a Levee holds only what is in transit, decrypts nothing and signs nothing in the Puddle's name.

The design draft with its open questions is Puddle's `docs/levee.md`. This part settles what the wire needs, above all
how a Levee fits **proving who answers** (part 02), which it would otherwise break.

## The problem receipts make

A sender checks that a receipt names the certificate *it* saw on the connection. Behind a Levee the sender's TLS
connection ends at the Levee, so:

1. the Puddle's answer, passed back by the Levee, would sign the Puddle's own certificate: the sender sees the Levee's
   and concludes **intercepted**;
2. while the Puddle is away the Levee must answer for it, and an answer the Puddle didn't sign counts as **not
   delivered**, so nothing would ever be handed to a Levee for an offline Puddle.

A Levee *is* a middle. The protocol's job is to make it an **announced** middle, so it is told apart from an attacker.

## The Levee's identity

A Levee has an Ed25519 key and id (part 01: `base32(SHA-256("puddle-levee-v1\n" || public key))`). It signs only its own
statements: custody receipts, and receipts on its own routes.

## Announcing a Levee

The Puddle's **address record** (part 03) gains one field:

    {"type":"address","id":..,"addresses":["levee.example.org:443"],"levee":{"id":"<levee id>","key":"<b64>"},"seq":N,"ts":..}

signed by the Puddle as always. It says: *at these addresses, this Levee answers for me.* Older Puddles ignore the field
(within-version additions are ignored, part 00), which only means they treat custody receipts as unproven and keep
retrying until the Puddle itself answers: slower, never wrong. A Puddle's link (`puddle://`) and first-contact
answers carry the same `levee` object, so a new friend learns it with the address.

## Receipts through a Levee: the Puddle vouches for its front

The Puddle connects to its Levee over TLS and so knows the Levee's certificate. When it answers a request that came
through the Levee, it signs its receipt (part 02) with `fp` = **the Levee's certificate**: the connection the sender
saw. The sender's check passes unchanged. An attacker can't get this: the Puddle signs only for the Levee it attached
to, and only it can make the signature.

Nothing in the receipt format changes; only which certificate `fp` names. A Levee MUST pass `X-Puddle-Receipt` back
untouched, and MUST NOT add one of its own to an answer that came from the Puddle.

## Custody receipts: the Levee takes it while the Puddle is away

When the Puddle is not attached, the Levee keeps one-way requests (events, address records, profiles, stream requests
and answers) and answers `202` with:

    X-Puddle-Custody: v1 for=<the Puddle's id>; levee=<levee id>; kid=<key id>; fp=<hex>; sig=<base64>

`sig` is Ed25519 by the Levee's key over:

    puddle-custody-v1
    <the Puddle's id>
    <the request's nonce>
    <hex SHA-256 of the request's body>
    <fp: the Levee's own certificate>

A sender that holds the Puddle's address record with that Levee's key accepts a good custody receipt as **delivered to
the Levee**: it stops retrying, and shows the owner "waiting at <the Puddle>'s Levee" rather than "delivered". The
Levee hands the spooled request, signature and all, to the Puddle when it attaches, and the Puddle checks it exactly as
if it had come directly. A custody receipt from a Levee the Puddle has not announced counts for nothing.

Requests that need an answer at once (a handshake, fetching a picture, a stream request's status) get no custody
receipt: the Levee answers the plain `404`, and the sender tries later as it would against an offline Puddle.

## The handshake behind a Levee

`GET /stream/v1/hello` is passed through while the Puddle is attached (the Puddle's receipt names the Levee's
certificate, as above). While it is not, the Levee answers `204` with a custody receipt over an empty body, which
proves the certificate is the announced Levee's, so the sender may send one-way requests, and only those, over it.

## Attaching (`/levee/v1`, sketched)

- `POST /levee/v1/attach`: the Puddle proves its identity (request signature, with its chain), and the operator's
  credential (an open question in `docs/levee.md`). The Levee answers with its id and key, signed by a receipt.
- The Puddle then holds a connection open (as a Pond member holds `/pond/v1/events`): `GET /levee/v1/inbox?after=<n>`
  hands over what was spooled and passes live requests through; the Puddle answers them with
  `POST /levee/v1/answer/<n>`.
- Outgoing: `POST /levee/v1/send` with the signed request to make; the Levee makes it, and spools it with retries if
  the far end is away, giving the Puddle a custody receipt of its own.

These are first sketches, to be settled when Levee is built; what is above them (the record field and the two receipts)
is what other implementations must agree on now.
