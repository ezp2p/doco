# Test vectors

Fixed inputs and the answer every implementation must give. Load them in your tests from a checkout of this repository
beside yours (`../protocol/vectors`).

| File | Spec | What an implementation must do |
|---|---|---|
| `devices.json` | 01, device ids | from each `seed`, make the Ed25519 key: its `public_key`, device `id` and six-digit `check` must match |
| `signatures.json` | 02 | `requests`: authenticate `header` for a request with that `method`, `target` and `body`, as the Puddle `checked_by` (or, with `accepts_any`, as anyone: recipient `*`), at `now_ms`, knowing only `sender`'s key; say `ok`, `reject` (the plain `404`) or `clock_skew`. `receipts`: check `header` as the caller of a request with that `nonce`, given `status`, `body` and the certificate it saw (`seen_fp`), knowing `responder`'s key; say `ok`, `missing`, `wrong_key`, `bad` or `other_cert` |

Keys are given as hex in the files; in the headers they are as `spec/02-signatures.md` says.

They were made by Puddle (`tests/protocol_vectors_test.c`, with `PUDDLE_WRITE_VECTORS=<this folder>`), which checks
itself against them on every build. Making them again changes every signature (new nonces, a new identity), so it is
done only when the protocol changes, and every implementation then checks the new ones.

Not covered yet: key chains (an inception and a rotation, and broken ones), address records, Pond ids, Levee custody
receipts. Each needs a vector file before a second implementation relies on it.
