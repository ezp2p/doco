# doco

Documentation about Puddles, Ponds, Levees and Streams.

Start with **[overview.md](overview.md)**: what each piece is and how they talk.

## Puddle

Its pages here are copies from the
[puddle repository](https://github.com/ezp2p/puddle), made with `scripts/sync-doco.sh` there; change them there, not here.

| | |
|---|---|
| [puddle/api/openapi.yaml](puddle/api/openapi.yaml) | The owner API: what the desktop app and phone clients call on their own Puddle. Objects in [puddle/api/objects/](puddle/api/objects/). |
| [puddle/api/federation.yaml](puddle/api/federation.yaml) | The stream API: what Puddles call on each other. Shapes shared by both in [puddle/api/wire/](puddle/api/wire/). |
| [puddle/docs/identity.md](puddle/docs/identity.md) | Who a Puddle is: keys from a recovery phrase, the key chain, rolling keys. |
| [puddle/docs/streams.md](puddle/docs/streams.md) | Becoming friends: asking for a stream, accepting, trust, blocks, invites. |
| [puddle/docs/groups.md](puddle/docs/groups.md) | Group chats between Puddles. |
| [puddle/docs/profiles.md](puddle/docs/profiles.md) | How an owner appears to others, and choosing who sees what. |
| [puddle/docs/files.md](puddle/docs/files.md) | Sending files, photos and videos (being built). |
| [puddle/docs/ponds.md](puddle/docs/ponds.md) | Being a member of Ponds: joining, following, what is kept. |
| [puddle/docs/objects.md](puddle/docs/objects.md) | The objects the owner API returns. |
| [puddle/docs/storage.md](puddle/docs/storage.md) | What a Puddle keeps on disk. |
| [puddle/docs/levee.md](puddle/docs/levee.md) | Levee, the relay a Puddle can sit behind (a design draft). |

## Pond

Text channels are built. These pages are copies from the [pond repository](https://github.com/ezp2p/pond), made with
`scripts/sync-doco.sh` there; change them there, not here.

| | |
|---|---|
| [pond/docs/design.md](pond/docs/design.md) | What a Pond is and does: invites, members, channels, events, versions, `pondctl`, the database. |
| [pond/api/pond.yaml](pond/api/pond.yaml) | The Pond API: what a member's Puddle calls. |

## Levee

Not built yet. Its design draft is [puddle/docs/levee.md](puddle/docs/levee.md).
