# Contributing to sentinel-contract

## Setup
```
cargo build --target wasm32-unknown-unknown --release
cargo test
cargo clippy --all-targets
```

## Before opening a PR
- Run the commands above and make sure they pass.
- Keep the PR scoped to one issue; reference it with `Closes #N`.
- Update README.md if you changed public behavior.

## Related repos
- [sentinel-backend](https://github.com/Stellar-Sentinel/sentinel-backend) — reads contract events and exposes them through its API; it does not currently submit transactions.
- [sentinel-frontend](https://github.com/Stellar-Sentinel/sentinel-frontend) — dashboard displaying screening results and backend-provided contract events.
