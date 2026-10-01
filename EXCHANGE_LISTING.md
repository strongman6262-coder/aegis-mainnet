# Aegis Protocol (AEGIS) - Exchange Integration & Listing Specification

## 1. Asset & Protocol Overview
- **Asset Name**: Aegis Protocol
- **Ticker Symbol**: `AEGIS`
- **Native Decimals**: `18`
- **Total Initial Supply**: `1,000,000,000 AEGIS` (1 Billion)
- **Chain ID**: `714261` (`0xae615`)
- **Consensus**: Tendermint-BFT Proof-of-Stake (2-second block finality)
- **Hashing Algorithm**: BLAKE3 / SHA3-256
- **Signature Scheme**: Ed25519 / Schnorr
- **Privacy Standard**: Dual-Key Stealth Addressing & Pedersen Commitments

---

## 2. Integration Interfaces

### A. Coinbase / Standard Rosetta API
Aegis implements the full Rosetta specification on port `8080`:
- **Network Identification**: `{"blockchain": "aegis", "network": "mainnet"}`
- **Account Balance**: `POST /account/balance` -> Returns liquid available and staked AEGIS balances.
- **Block Fetching**: `POST /block` -> Retrieves verified blocks with deterministic state roots.
- **Construction & Broadcast**: `POST /construction/submit` -> Broadcasts serialized transactions.

### B. Standard JSON-RPC 2.0 (Ethereum/Web3 Compatible)
Endpoints exposed on port `8545`:
- `aegis_blockNumber` / `eth_blockNumber`: Returns current chain height.
- `aegis_getBalance` / `eth_getBalance`: Retrieves 18-decimal base unit balance.
- `aegis_getBlockByNumber` / `eth_getBlockByNumber`: Returns block data, transactions, and execution receipts.
- `aegis_getTransactionReceipt` / `eth_getTransactionReceipt`: Confirms transaction inclusion & gas consumed.
- `aegis_sendRawTransaction` / `eth_sendRawTransaction`: Broadcasts bincode/hex encoded signed transactions.
- `aegis_nodeInfo`: Returns health, active validator count, and total staked volume.

---

## 3. Deposit & Withdrawal Workflow for Exchanges

### Deposits (Hot Wallet Ingestion)
1. Generate an Ed25519 deposit address for each user: `0x<32_byte_hex_address>`.
2. Monitor new blocks via `aegis_getBlockByNumber(height, true)` or subscribe via WebSocket.
3. For each transaction with `PublicTransfer` matching a registered deposit address, confirm after **2 block confirmations** (4 seconds for instant finality).
4. Credit user balance.

### Withdrawals
1. Create a `TransactionPayload::PublicTransfer { recipient, amount }`.
2. Sign with hot wallet private key via Ed25519.
3. Submit using `aegis_sendRawTransaction(rawHex)`.
4. Poll `aegis_getTransactionReceipt(txHash)` for `ExecutionStatus::Success`.

---

## 4. Staking & Node Running Commands
```bash
# Build the production node binary
cargo build --release -p aegis-node

# Run full node with JSON-RPC enabled
./target/release/aegis-node --config config.toml

# Run Validator / Mining Node
./target/release/aegis-node --config config.toml --validator
```
