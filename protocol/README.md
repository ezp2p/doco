# The Puddle Protocol

What every ezp2p program says on the wire, written down once, so programs built apart, in their own code, upgraded at
their own pace, still understand each other.

| Program | What it is | Speaks |
|---|---|---|
| **Puddle** (`ezp2p/puddle`) | Self-hosted chat with an identity, on the owner's computer | everything below |
| **Phone client** (inside `ezp2p/puddle`) | A thin client of its owner's Puddle | devices |
| **Pond** (`ezp2p/pond`) | A community server, like a Discord server. Its channels live on the Pond; Puddles cache them | identity, signatures, ponds |
| **Levee** (`ezp2p/levee`) | A stable public front for one Puddle (or a few): spools while it is offline, keeps its address when it moves | identity, signatures, levee |
| **Drip** (`ezp2p/drip`) | P2P social media (not designed yet) | identity, signatures, events; see `spec/07-drip.md` |

## What is here

- [`decisions.md`](decisions.md): **why this exists, and what it is not** (no shared library yet). Read this first.
- `spec/`: the protocol, one part per file. Where a part says MUST, it is a rule an implementation breaks at its peril.
  - [`00-overview.md`](spec/00-overview.md): the principles every part follows, and how the protocol is versioned
  - [`01-identity.md`](spec/01-identity.md): ids, keys, the key chain
  - [`02-signatures.md`](spec/02-signatures.md): request signatures and answer receipts, byte for byte
  - [`03-streams.md`](spec/03-streams.md): Puddle to Puddle
  - [`04-devices.md`](spec/04-devices.md): a phone and its Puddle
  - [`05-ponds.md`](spec/05-ponds.md): a Puddle and a Pond
  - [`06-levee.md`](spec/06-levee.md): a Puddle behind a Levee, and what that does to receipts
  - [`07-drip.md`](spec/07-drip.md): what Drip can take from all this
- `vectors/`: test vectors: fixed keys, ids, signed requests and receipts, each with the answer an implementation must
  give. Every implementation's tests load them (Puddle: `protocol_vectors` test). A change to the wire changes a vector.

## Status

This is the first cut (2026-11-12), written from what Puddle and Pond do today. Puddle's `docs/streams.md`,
`docs/identity.md` and `api/federation.yaml`, and Pond's `docs/design.md` and `api/pond.yaml`, still hold the fuller
text; as parts settle here, those documents point here instead of repeating it.

| Part | Puddle | Pond | Levee | Drip |
|---|---|---|---|---|
| identity, chains | built | built (its own code) | not built | - |
| request signatures | built | built (its own code) | not built | - |
| receipts | built (phase 6) | **not yet**: a Puddle pins a Pond's certificate | designed here | - |
| streams | built | - | passes them on | - |
| devices | built (phase 7) | - | - | - |
| ponds | built (member side) | built | passes them on | - |
| levee | designed here | - | designed here | - |
