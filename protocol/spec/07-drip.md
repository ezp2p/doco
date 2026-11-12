# 07. Drip: what P2P social media can take from this

*Notes, not a design. Drip is not designed yet.*

Drip is to be peer-to-peer social media: people post, others follow them. Most of what it needs already exists here,
and reusing it means one identity for chatting and posting, and friends found in one place are known in the other.

- **Identity (part 01), as it is.** A Drip author is a Puddle identity: the same user id, the same chain, the same
  rolling and recovery. Following someone is knowing their id and an address, exactly as a stream starts.
- **Posts are signed events (part 03).** An event is already signed by its author over its exact bytes and names the key
  (`kid`), so it verifies forever, wherever it is copied from. A post is an event of a new kind in a per-author log.
  Because each post proves itself, followers can pass posts on to each other, and a post is genuine however it arrived.
  Public posts are not encrypted; posts for followers only can be encrypted to them as group events are.
- **Following is a scoped stream.** Puddle already has `scoped` streams (group-only connections); a `follow` scope is
  the same: a signed relationship that carries posts one way and nothing else.
- **Levees (part 06) are what makes it work.** Social media needs an always-on address to fetch from. A Levee is
  exactly that: it can hold its Puddle's public log and answer followers' fetches while the Puddle is off, with every
  post still signed by the author, so the Levee serves posts but can't forge them.
- **Ponds (part 05) as public spaces.** Hashtags and public timelines are places, and places are what Ponds are.

What Drip adds to the protocol would be: an event kind for posts, a `follow` stream scope, and a read-only "fetch an
author's log after `n`" route that a Levee can answer. Each is an addition within a version (part 00).
