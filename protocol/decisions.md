# Decisions

## Do we need a separately defined protocol? Yes: a specification and test vectors. Not (yet) a shared library.

*Decided 2026-11-12, to be reviewed by the owner.*

### Why a protocol of its own

- **There are already two implementations.** Pond "shares no code with Puddle" (its README): it checks key chains and
  request signatures in its own `src/identity.cpp` and `src/reqsig.cpp`, written from Puddle's documentation. Levee
  will be a third and Drip a fourth. Programs that must agree byte for byte need one text that says what the bytes are,
  owned by none of them.
- **Puddle's documents are not that text.** They mix the wire with how Puddle happens to be built (its folders, its
  outbox schedule, its app). A Pond or Levee author has to dig the rules out, and a Puddle change can move a rule
  without anyone noticing that the Pond now disagrees.
- **The parts now depend on each other.** Two examples from this week:
  - Proving who answers (receipts, Puddle phase 6) changes what every responder must send, Ponds included, and it
    breaks a Puddle behind a Levee unless the Levee is designed for it (`spec/06-levee.md`). That is a cross-project
    decision, and it needed a cross-project place to be made.
  - A Puddle pins a Pond's TLS certificate because Ponds don't sign receipts, so a renewed certificate cuts every
    member off. The fix (Ponds sign receipts) is a protocol change both sides must make.
- **Versioning needs one owner.** Pond already versions by path (`/pond/v1`) and advertises `protocol` and
  `features`. Puddle versions by path (`/stream/v1`). Both need the same rules for what may change within a version;
  `spec/00-overview.md` states them once.

### Why test vectors

A specification can be misread. Vectors can't: fixed inputs, the exact expected outputs, and the answer a checker must
give (accept, or which refusal). Each implementation's own tests load `vectors/*.json` from a checkout beside it
(Puddle's `protocol_vectors` test looks in `../protocol`, as Pond's end-to-end test looks for `../puddle`), so any
disagreement fails a build instead of a conversation. The vectors were made by Puddle's code, which is the reference
until something here says otherwise.

### Why not a shared library, yet

- **The programs are built very differently on purpose.** Puddle is plain CMake with OpenSSL and libyaml and no other
  dependencies, so it builds anywhere, a phone included. Pond is built on Drogon with MariaDB, Redis, jsoncpp and
  yaml-cpp. A shared library would have to fit both build systems and both sets of dependencies, and would tie their
  release cycles together: exactly the coupling the versioning rules exist to avoid.
- **The shared part is small and stable.** Ids, the key chain, request signatures, receipts: a few hundred lines that
  rarely change. Vectors guard that well; a library mostly saves typing.
- **Two implementations agreeing is a feature.** When Pond's code, written from the text alone, passes the vectors,
  the text is proven complete. A shared library would hide gaps in the text.

**When to revisit:** when Levee is built. Levee must check chains, request signatures and receipts and sign custody
receipts (`spec/06-levee.md`); if it is C++ like the others, extracting `libpuddle-proto` (C-callable, OpenSSL only:
ids, chains, request signatures, receipts) from Puddle's `src/core/identity.cpp` and `src/http/reqsig.cpp` becomes worth
it, as three copies of the same few hundred lines is one too many. The vectors stay either way.

### Where this lives

`ezp2p/protocol` as its own repository, so it is versioned and reviewed apart from any one program. Each program's
`docs/protocol.md` says which parts it implements and links here. (Made locally on 2026-11-12; it still needs a
repository on GitHub, which only the owner can create.)

## Receipts through a Levee: the Puddle vouches for its front

See `spec/06-levee.md`. A Levee terminates TLS for the Puddle behind it, so a receipt over the Puddle's own certificate
would look like interception. Decided: a Puddle answering through its Levee signs its receipt over **the Levee's**
certificate (it knows it: it connected to it), and its address record names the Levee's key. A Levee answering for an
offline Puddle signs a **custody receipt** with its own key, which senders accept only for a Levee that Puddle has
announced. Nothing in the existing receipt changes; one new header and one new address-record field are added.

## Ponds should sign receipts

See `spec/05-ponds.md`. Proposed, not built: Ponds answer members with the same `X-Puddle-Receipt` Puddles use, signed
by the Pond's key. Then a Puddle stops pinning a Pond's certificate, and a Pond can renew its certificate (Let's
Encrypt, say) without cutting everyone off. Advertised as the feature `receipts`, so Puddles keep pinning Ponds that
don't have it.
