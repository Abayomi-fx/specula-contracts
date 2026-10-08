# Stellar Sentinel — Soroban Contract

[![CI](https://github.com/Stellar-Sentinel/sentinel-contracts/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Stellar-Sentinel/sentinel-contracts/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

The Soroban smart contract component of Stellar Sentinel. It provides an on-chain registry for an administrator, authorized monitoring agents, and a risk threshold. An authorized agent can emit a risk flag for an address when its score meets the configured threshold.

## What it does

- `initialize(admin, default_threshold)` sets the administrator and initial threshold once. The admin must authorize the call.
- `authorize_agent(admin, agent)` lets the stored admin authorize an agent address.
- `revoke_agent(admin, agent)` removes an agent's flagging permission.
- `set_threshold(admin, threshold)` updates the accepted threshold; values must be in the inclusive `0..=100` range.
- `is_agent(agent)` reports whether an address is authorized.
- `get_threshold()` returns the configured threshold.
- `flag_anomaly(agent, subject, score)` requires agent authorization, validates the score is in `0..=100`, checks it against the threshold, records the latest flag for the subject, and publishes an event when the threshold is met.
- `get_latest_flag(subject)` returns the latest flag record, or `None` when the subject has not been flagged.

The flag event retains the `flagged` topic and publishes `(agent, subject)` as event topics with `score` as its value. This stable event schema supports off-chain indexing. The contract also stores one latest record per subject containing the agent, score, ledger sequence, and ledger timestamp; this is current state, not a complete history. Persistent records are extended when written or read and remain subject to Soroban network TTL limits.

## How it integrates with Stellar Sentinel

Soroban is Stellar's smart contract platform. This contract is intended to be the on-chain destination for decisions made by the off-chain Stellar Sentinel backend. An authorized agent would submit a score, and Soroban would enforce authorization and the configured threshold before emitting the event. The backend transaction client and the frontend event reader are not implemented yet, so the contract currently operates independently.

```text
Stellar activity → backend scoring → authorized agent transaction → this Soroban contract
                                                               └── flagged event
```

The current backend score endpoint accepts caller-supplied metrics and does not send transactions. See [sentinel-backend](https://github.com/Stellar-Sentinel/sentinel-backend) and [sentinel-frontend](https://github.com/Stellar-Sentinel/sentinel-frontend) for the other components.

## Build and test

Requires Rust and the `wasm32-unknown-unknown` target.

```bash
rustup target add wasm32-unknown-unknown
cargo build --target wasm32-unknown-unknown --release
cargo test
cargo clippy --all-targets -- -D warnings
```

The release Wasm artifact is written under `target/wasm32-unknown-unknown/release/`.

## Security and current limitations

- Agent authorization is a single role: authorized agents can all submit flags. There are no separate monitor and responder roles.
- The latest flag record is overwritten each time a subject is flagged; index events off-chain for complete history.
- Scores and thresholds use a 0 to 100 scale. This range check does not establish that a score is accurate or that the backend's scoring policy is sound.
- Instance configuration and subject records follow Soroban TTL rules. Integrations should account for archival and restoration if records expire under network policy.
- No deployment address is configured in this repository. Deploy and initialize separately for each Stellar network, and authorize only trusted agent addresses.

Do not treat testnet deployment or the prototype backend score as production risk controls. A production deployment needs a reviewed scoring policy, secure agent key management, monitoring, and a contract upgrade and recovery plan.

## Contributing and license

See [CONTRIBUTING.md](CONTRIBUTING.md). Licensed under the [MIT License](LICENSE).
