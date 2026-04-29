# Method Coverage Matrix

Quick reference for gRPC method support across releases.

---

## MVP Methods (v0.1.0)

| Method | Status | Purpose |
|--------|--------|---------|
| `GetLatestBlock` | ✅ Implemented | Retrieve latest block height and hash |
| `GetBlockRange` | ✅ Implemented | Fetch multiple blocks by height range |
| `GetLatestTreeState` | ✅ Implemented | Get current commitment tree state |

**Test Coverage:** [cases/mvp/](../cases/mvp/) (4 test cases)

---

## Later / Full-Mode Methods

| Method | Status | Purpose |
|--------|--------|---------|
| `GetTreeState` | 🔜 Planned | Retrieve tree state at specific height |
| `GetSubtreeRoots` | 🔜 Planned | Subtree root queries for proof generation |
| `GetMempoolTx` | 🔜 Planned | Access unconfirmed transactions in mempool |
| `GetTaddressTransactions` | 🔜 Planned | Query transparent address transactions |

**Roadmap:** v0.2.0+ (planned after MVP)

---

**Last Updated:** April 2026
