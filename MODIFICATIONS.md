# MODIFICATIONS

Fork of [HNScan/hnscan-backend](https://github.com/HNScan/hnscan-backend) (MIT).
Branch `revival-p1` from upstream master `0aff37e` (2020-03-31). Explorer
revival phase 1: dependency resurrection + minimal API compatibility, no
feature work.

## Dependency resurrection (Node 22 LTS, hsd v6.x)

`package.json` dependencies rewritten to mirror hsd v6.1.1's own dependency
matrix — the plugin runs inside the hsd process, so the shared `b*` libraries
must be the same lineage to avoid duplicate incompatible module instances:

| Dependency | Was | Now |
|---|---|---|
| hsd | `github:handshake-org/hsd` (floating master) | `github:handshake-org/hsd#v6.1.1` (pinned) |
| bcrypto | ^3.0.1 | ~5.4.0 |
| bdb | ^1.1.2 | ~1.4.0 |
| bevent | ^0.1.2 | ~0.1.5 |
| blgr | ^0.1.2 | ~0.2.0 |
| bmutex | ^0.1.5 | ~0.1.6 |
| bsert | 0.0.5 | ~0.0.12 |
| bstring | ^0.3.4 | ~0.3.5 |
| bufio | ^1.0.3 | ~1.2.0 |
| bval | ^0.1.4 | ~0.1.8 |
| bweb | ^0.1.4 | ~0.2.0 |
| geoip-lite | ^1.3.8 | ^1.4.10 |
| path | ^0.12.7 | REMOVED (Node builtin; the npm package was a mistake) |

`yarn.lock` removed (npm is the package manager for the revival).

## Code changes

- `lib/http.js` — `/summary` response gains snake_case aliases
  (`chain_work`, `unconfirmed_size`, `registered_names`): the frontend at its
  2020-06 head reads snake_case; camelCase originals stay for compatibility.
- `lib/hnscan.js` — block serializer gains snake_case aliases
  (`merkle_root`, `witness_root`, `tree_root`, `reserved_root`,
  `previous_hash`, `next_hash`, `mined_by`, `num_tx`, `extra_nonce`);
  tx serializer gains `tx_id` (alias of `hash`). Same reason.

## Runbook facts (discovered, not changed)

- The host hsd MUST run with `--index-tx --index-address` from block 0 —
  `getMeta` returns null otherwise and `/names/:name/history` (and address
  endpoints) fail. hsd cannot retroactively enable TX indexing on an
  existing chain directory.
- Plugin flags: `--plugins=<path-to>/lib/index.js --hnscan-http-port=<port>`
  (default port 8080; our runbook uses 8083).
