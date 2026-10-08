# Stellar Sentinel — Soroban Contract

[![CI](https://github.com/Stellar-Sentinel/sentinel-contracts/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Stellar-Sentinel/sentinel-contracts/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

The Soroban smart contract component of Stellar Sentinel. It provides an on-chain registry for an administrator, authorized monitoring agents, and a risk threshold. An authorized agent can emit a risk flag for an address when its score meets the configured threshold.

## What it does

- `initialize(admin, default_threshold)` sets the administrator and initial threshold once. The admin must authorize the call.
- `authorize_agent(admin, agent)` lets the stored admin authorize an agent address.
- `is_agent(agent)` reports whether an address is authorized.
- `get_threshold()` returns the configured threshold.
- `flag_anomaly(agent, subject, score)` requires agent authorization, checks the score against the threshold, and publishes an event when the threshold is met.

The flag event uses the `flagged` topic and publishes `(agent, subject)` as event topics with `score` as its value. Flags are events only; the contract does not persist a history or provide an event query method.

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

- Agent authorization is a single role: authorized agents can all submit flags. There are no separate monitor and responder roles or agent revocation method.
- Thresholds are supplied by the admin at initialization and cannot currently be changed afterward.
- A flag is not stored as contract state, so applications that need history must index Soroban events off-chain.
- The contract does not validate that the score is within a particular range; it only checks that it meets the stored threshold.
- No deployment address is configured in this repository. Deploy and initialize separately for each Stellar network, and authorize only trusted agent addresses.

Do not treat testnet deployment or the prototype backend score as production risk controls. A production deployment needs a reviewed scoring policy, secure agent key management, monitoring, and a contract upgrade and recovery plan.

## Contributing and license

See [CONTRIBUTING.md](CONTRIBUTING.md). Licensed under the [MIT License](LICENSE).
