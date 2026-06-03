# TrustLedger — Core

> Soroban indexing engine, ZK proof circuits, and GraphQL compliance API for Real-World Assets on Stellar.

This repository contains the full backend infrastructure for TrustLedger — the Zephyr-based on-chain indexer, Space and Time zero-knowledge Proof of SQL circuits, and the GraphQL delivery layer that serves institutional ERP platforms and the TrustLedger Dashboard.

<!-- > **Related repository:** [`trustledger-dashboard`](../trustledger-dashboard) — Institutional compliance and audit frontend. -->

---

## Why This Exists

The rapid growth of tokenized yield-bearing assets on Stellar requires enterprise-grade auditing infrastructure that standard RPC setups cannot support. TrustLedger Core addresses key backend data requirements:

- **Standard RPC providers** are optimized for current ledger states and do not natively support complex, historical, or aggregate SQL analytical queries.
- **Regulators, custodians, and institutional funds** require immutable proof that historical asset distributions and compliance metrics are untampered with.
- **Centralized indexers** reintroduce third-party trust assumptions, undermining the core benefits of a public, decentralized ledger.
- **Scaling applications** require lightweight, sandboxed data-processing solutions to avoid high database infrastructure costs.

---

## Repository Structure

```
.
├── zephyr-app/                        # Rust indexer logic compiling to WASM for Mercury ZephyrVM
│   ├── src/
│   │   ├── lib                     # Indexer entry point and event handler definitions
│   │   └── filters                 # Ledger event filtering and transformation logic
│   └── Cargo.toml
├── zk-proofs/                         # Zero-knowledge proof circuit definitions
│   ├── circuits/                      # Space and Time Proof of SQL circuit schemas
│   └── scripts/                       # Proof generation and verification helper scripts
├── api/                               # GraphQL query server
│   ├── src/
│   │   ├── schema/                    # GraphQL type definitions and resolvers
│   │   └── middleware/                # Auth, rate limiting, and compliance guards
│   ├── package.json
│   └── tsconfig.json
├── docs/                              # Compliance architecture sheets, schema definitions,
│                                      # and audit verification guides
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                     # Lint, test, and WASM build on every push and PR
│   │   ├── deploy-indexer.yml         # Deploy compiled WASM to Mercury ZephyrVM on merge to main
│   │   └── proof-regression.yml       # Nightly ZK proof circuit regression tests against Testnet
│   └── SECURITY.md
├── .husky/
│   ├── pre-commit                     # Runs cargo fmt, clippy, and ESLint before every commit
│   └── pre-push                       # Runs full test suite before pushing to remote
├── Dockerfile                         # Multi-stage build for the GraphQL API service
├── docker-compose.yml                 # Local orchestration: API, PostgreSQL, and Redis cache
├── .env.example
└── README.md
```

---

## Quick Start

### Prerequisites

- Rust stable + WASM toolchain (`wasm32-unknown-unknown` target)
- Node.js 22+, npm 10+
- Mercury CLI / Zephyr SDK
- Space and Time API credentials
- Docker + Docker Compose

### Environment Setup

```bash
cp .env.example .env
```

```env
# Mercury / Zephyr
MERCURY_API_KEY=your_mercury_api_key
MERCURY_NETWORK=mainnet                  # or testnet

# Space and Time
SXT_API_KEY=your_sxt_api_key
SXT_API_ENDPOINT=https://api.spaceandtime.io

# GraphQL API
API_PORT=4000
JWT_SECRET=your_jwt_secret

# PostgreSQL
POSTGRES_USER=trustledger
POSTGRES_PASSWORD=your_db_password
POSTGRES_DB=trustledger_core

# Redis
REDIS_URL=redis://localhost:6379
```

---

## Development

### Install API Dependencies and Set Up Husky

```bash
cd api
npm install
npm run prepare          # Initialises Husky git hooks
```

Husky enforces the following gates automatically:

| Hook | Actions |
|---|---|
| `pre-commit` | `cargo fmt --check`, `cargo clippy`, ESLint on `api/` |
| `pre-push` | Full Rust test suite + GraphQL API integration tests |

---

### Build Zephyr Indexing WASM

```bash
cd zephyr-app
cargo build --target wasm32-unknown-unknown --release
```

### Deploy Indexer to Mercury ZephyrVM

```bash
mercury-cli deploy \
  --wasm zephyr-app/target/wasm32-unknown-unknown/release/rwa_indexer.wasm \
  --network mainnet
```

### Execute a Verified Proof of SQL Query

```bash
sxt-cli query \
  --sql "SELECT sum(yield_accrued) FROM spiko_rwa_holders" \
  --verify-zk
```

---

## Running Locally with Docker

Spin up the full local stack — GraphQL API, PostgreSQL, and Redis — with a single command:

```bash
docker compose up --build
```

| Service | Local URL |
|---|---|
| GraphQL API | `http://localhost:4000/graphql` |
| PostgreSQL | `localhost:5432` |
| Redis | `localhost:6379` |

To run in detached mode:

```bash
docker compose up --build -d
```

To tear down and remove volumes:

```bash
docker compose down -v
```

---

## CI/CD Workflows

| Workflow | Trigger | Actions |
|---|---|---|
| `ci.yml` | Every push and pull request | Cargo lint, Clippy, WASM build, API unit tests |
| `deploy-indexer.yml` | Merge to `main` | Builds and deploys compiled WASM to Mercury ZephyrVM |
| `proof-regression.yml` | Nightly schedule | Runs ZK circuit regression tests against Testnet ledger data |

---

## Running Tests

### Rust (Indexer + ZK Circuits)

```bash
cargo test --all-targets
```

### GraphQL API

```bash
cd api
npm run test
npm run test:integration
```

### All Targets

```bash
npm run test:all          # Runs Rust and Node test suites sequentially
```

---

## Security

To report a vulnerability, open a github issue.

All indexed datasets are cryptographically pinned. Queries against the GraphQL API generate verified Proof of SQL proofs through Space and Time, ensuring data integrity guarantees for institutional consumers without requiring trust in TrustLedger infrastructure.

---

<!-- ## License

<!-- Add license here -->
