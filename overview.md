# Overview

Puddles is a chat platform with no server in the middle. Everyone runs their own.

## The pieces

- **Puddle**: the app on a person's own computer. It keeps their identity, friends, chats and files, and holds a port
  open so other people's Puddles can reach it. It comes as a desktop app (Qt) and as `puddled`, the same thing with no
  window. *Built.*
- **Phone client**: an app on a phone that connects back to its owner's own Puddle. It hosts nothing. *Not built.*
- **Stream**: a trusted connection between two Puddles: what two people have once they are friends. Nothing passes
  between two Puddles without one, apart from asking for one.
- **Pond**: a community server, like a Discord server: channels, many members, and later roles. Someone runs it (`pondd`,
  with MariaDB, and `pondctl` to manage it); people join from their Puddle with a `pond://` invite link. A separate
  program. *Text channels built.*
- **Levee**: a relay for a Puddle that cannot accept connections from the internet. The Puddle connects out to it and
  everything goes through it; it holds what arrives while the Puddle is offline and can read none of it. *A design
  draft.*

```
 owner's app ──(owner API, HTTPS)──► Alice's Puddle ◄──(stream API, signed HTTPS)──► Bob's Puddle
 (desktop, phone)                          ▲                                           ▲
                                           │                                           │
                                           └──(Pond API, signed HTTPS)──► a Pond ◄─────┘
                                                                         (later: Levees)
```

## Three APIs

| | Who calls it | Spec |
|---|---|---|
| **Owner API** (`/api/v1`) | The owner's own apps, with the owner token | [puddle/api/openapi.yaml](puddle/api/openapi.yaml) |
| **Stream API** (`/stream/v1`) | Other Puddles, each request signed by the sender's key | [puddle/api/federation.yaml](puddle/api/federation.yaml) |
| **Pond API** (`/pond/v1`) | A member's Puddle, signed the same way | [pond/api/pond.yaml](pond/api/pond.yaml) |

Both share a few rules:

- **Nothing is public.** A request that fails any check gets the same `404` as a path that does not exist, so a stranger
  learns nothing, not even that this is a Puddle.
- **Only GET and POST.** GET reads; POST acts, with the verb last in the path (`/users/{user}/block`).
- The owner API changes as the apps need. The stream API has to keep working with older Puddles, so it only grows.

## What travels between Puddles

- **Identity:** a key chain that starts from a recovery phrase and can be rolled to new keys
  ([identity](puddle/docs/identity.md)).
- **Streams:** asking, accepting, trusting a changed certificate, closing ([streams](puddle/docs/streams.md)).
- **Events:** messages, edits and deletions, signed by their author and delivered to each person in the chat; group
  events say who is in a group ([groups](puddle/docs/groups.md)).
- **Profiles:** the one the owner chose for each place ([profiles](puddle/docs/profiles.md)).
- **Files:** fetched in checked chunks from whoever has them; sent by itself once, and again only with the holder's yes ([files](puddle/docs/files.md)).

## What travels between a Puddle and a Pond

The Puddle always makes the connection, so it works from behind NAT. It joins with an invite and then holds one request
open for the Pond's events: new messages, channels and members arrive as they happen. It keeps the newest 200 messages of
each channel and asks the Pond for anything older. The Pond keeps everything else ([Pond design](pond/docs/design.md),
[the Puddle's side](puddle/docs/ponds.md)).
