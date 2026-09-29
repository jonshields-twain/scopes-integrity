# scopes-integrity

Public integrity ledger for [Scopes](https://rate-scope.vercel.app). Each UTC day, Scopes
computes a Merkle root over that day's append-only record (price snapshots, flagged
divergences, reads, model reads, resolutions and corrections) and anchors it to the Bitcoin
blockchain via [OpenTimestamps](https://opentimestamps.org). This repository mirrors every
day's root and its `.ots` proof, so the record can be proven to have existed on that date and
to be unaltered since, without trusting Scopes.

## Layout

- `digests/<YYYY-MM-DD>.merkle` — the day's Merkle root (lowercase hex, RFC 6962).
- `digests/<YYYY-MM-DD>.merkle.ots` — the OpenTimestamps proof for that root.
- `index.jsonl` — one line per day: `{date, merkle_root, row_count, ots_status, ...}`.

## Verify a day

1. Recompute the root from a Scopes data export with the reference verifier
   (`scripts/verify-integrity.mjs` in the Scopes repo); it must equal `digests/<date>.merkle`.
2. Verify the timestamp: `ots verify digests/<date>.merkle.ots` (with the `.merkle` file
   present) confirms the root was committed to Bitcoin at or before the attested block time.

Canonicalization is frozen as **v1** (RFC 6962 Merkle); the full rules are in
`docs/integrity-canonicalization-v1.md` in the Scopes repo. See
[scopes.com/integrity](https://rate-scope.vercel.app/integrity) for the live ledger.

Backfilled days (before same-day anchoring went live) are labelled `anchored_retroactively`:
the record existed from that date, but its proof was created when backfill ran.
