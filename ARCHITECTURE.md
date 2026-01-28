# Architecture Overview

**Golden Triangle Method**  
**Designed & Maintained by magic**

---

## 🏗️ System Architecture

The Golden Triangle Method implements a layered architecture combining on-chain (Layer 1) and off-chain (Layer 2) components:

```
┌─────────────────────────────────────────────────────────────┐
│                    USER LAYER                               │
│              Web Frontend / Mobile Apps                     │
└────────────────┬────────────────────────────────────────────┘
                 │
                 │ HTTP/WebSocket
                 ▼
┌─────────────────────────────────────────────────────────────┐
│               SEQUENCER LAYER (L2) - RUST                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────┐        ┌───────────────┐               │
│  │  Transaction  │───────►│   ZK Proof    │               │
│  │     Pool      │        │   Generator   │               │
│  │               │        │   (Groth16)   │               │
│  └───────────────┘        └───────────────┘               │
│         │                         │                        │
│         │                         │                        │
│         ▼                         ▼                        │
│  ┌───────────────┐        ┌───────────────┐               │
│  │     State     │        │    Batch      │               │
│  │   Manager     │◄───────│  Coordinator  │               │
│  │               │        │               │               │
│  └───────────────┘        └───────────────┘               │
│         │                         │                        │
│         │                         │ Submit Proofs          │
└─────────┼─────────────────────────┼────────────────────────┘
          │                         │
          │ Event Stream            │
          │                         ▼
┌─────────────────────────────────────────────────────────────┐
│              BLOCKCHAIN LAYER (L1) - SUI                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────┐      ┌──────────────────────┐    │
│  │   Privacy Pool       │      │  Groth16 Verifier    │    │
│  │  (Move Contract)     │◄────►│  (Move Contract)     │    │
│  │                      │      │                      │    │
│  │  - Deposits          │      │  - Verify Proofs     │    │
│  │  - Withdrawals       │      │  - BN254 Pairing     │    │
│  │  - State Updates     │      │  - Commitment Check  │    │
│  └──────────────────────┘      └──────────────────────┘    │
│           │                              │                 │
│           ▼                              ▼                 │
│  ┌──────────────────────┐      ┌──────────────────────┐    │
│  │   Merkle Tree        │      │  Nullifier Set       │    │
│  │   (Commitments)      │      │  (Prevent Double-    │    │
│  │                      │      │   Spending)          │    │
│  └──────────────────────┘      └──────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔄 Transaction Flow

### 1. Deposit Flow (Shielding Assets)

```
User
  │
  │ 1. Generate commitment C = H(amount, secret, randomness)
  ▼
Frontend
  │
  │ 2. Create deposit transaction
  ▼
Move Contract (L1)
  │
  │ 3. Lock SUI in privacy pool
  │ 4. Add commitment C to Merkle tree
  │ 5. Emit DepositEvent
  ▼
Sequencer (L2)
  │
  │ 6. Index event
  │ 7. Update L2 state
  ▼
User notified: Deposit confirmed
```

### 2. Withdrawal Flow (Unshielding Assets)

```
User
  │
  │ 1. Select commitment to spend
  │ 2. Generate nullifier N = H(secret, index)
  ▼
Sequencer (L2)
  │
  │ 3. Generate ZK proof π proving:
  │    - Knowledge of secret
  │    - Commitment exists in Merkle tree
  │    - Nullifier correctly derived
  │    - No double-spending
  │
  │ Time: ~2-5 seconds
  ▼
Move Contract (L1)
  │
  │ 4. Verify proof π
  │ 5. Check nullifier N not used
  │ 6. Mark nullifier as spent
  │ 7. Transfer SUI to recipient
  │ 8. Emit WithdrawEvent
  ▼
User receives SUI
```

---

## 🔺 The Golden Triangle Components

### MATH (The Shield) 🛡️

**Location**: Embedded in Sequencer (Rust) and Verifier (Move)

**Responsibilities**:
- Zero-knowledge proof generation (Groth16)
- Cryptographic guarantees of privacy
- Commitment and nullifier generation
- Merkle tree proof computations

**Key Algorithms**:
```
Commitment:   C = Poseidon(amount, recipient, randomness)
Nullifier:    N = Poseidon(secret, index)
Merkle Proof: P = [H₀, H₁, ..., H₂₀] proving C ∈ Tree
ZK Circuit:   π ← Groth16.Prove(C, N, P, secret)
```

---

### MOVE (The Logic) 🎯

**Location**: Sui blockchain (Layer 1)

**Smart Contracts**:

#### Privacy Pool
```move
struct PrivacyPool has key {
  id: UID,
  merkle_root: vector<u8>,           // Current tree root
  nullifiers: Table<vector<u8>, bool>, // Spent nullifiers
  total_deposits: u64,                // Total locked value
}
```

**Functions**:
- `deposit()`: Lock assets, add commitment
- `withdraw()`: Verify proof, release assets
- `update_root()`: Batch commitment insertions

**Security Properties**:
- ✓ No reentrancy (Move linear types)
- ✓ No double-spending (nullifier tracking)
- ✓ Asset conservation (type system guarantees)

---

### RUST (The Engine) ⚙️

**Location**: Off-chain sequencer (Layer 2)

**Components**:

#### 1. Transaction Batcher
```rust
struct TransactionBatcher {
  pending_txs: VecDeque<Transaction>,
  batch_size: usize,
  batch_timeout: Duration,
}
```

#### 2. Proof Generator
```rust
async fn generate_proof(tx: Transaction) -> Result<Proof> {
  // Build witness from transaction
  let witness = build_witness(&tx)?;
  
  // Generate Groth16 proof (CPU-intensive)
  let proof = groth16::create_proof(witness, &PARAMS)?;
  
  Ok(proof)
}
```

#### 3. State Manager
```rust
struct StateManager {
  merkle_tree: IncrementalMerkleTree,
  l2_balances: HashMap<Address, u64>,
  pending_withdrawals: Vec<WithdrawalRequest>,
}
```

#### 4. L1 Bridge
```rust
async fn submit_to_l1(proof: Proof) -> Result<TxDigest> {
  // Submit proof to Sui blockchain
  let tx = sui_client
    .execute_withdraw(proof)
    .await?;
    
  Ok(tx.digest)
}
```

**Performance Optimizations**:
- Parallel proof generation (Rayon)
- Asynchronous L1 coordination (Tokio)
- Memory-efficient Merkle tree (sparse storage)
- Batch proof aggregation (future)

---

## 🔐 Security Model

### Threat Mitigation

| Attack Vector | Mitigation | Layer |
|---------------|------------|-------|
| **Double-spending** | Nullifier tracking | L1 (Move) |
| **Invalid proofs** | On-chain verification | L1 (Move) |
| **Malicious sequencer** | All proofs verified on-chain | L1 (Move) |
| **Blockchain analysis** | Zero-knowledge hiding | Cryptography |
| **Front-running** | Commitment scheme prevents correlation | Architecture |
| **Reentrancy** | Move linear types | L1 (Move) |
| **Integer overflow** | Checked arithmetic | L1 (Move) |

### Trust Assumptions

**Trusted**:
- ✓ Sui blockchain liveness and finality
- ✓ Groth16 cryptographic assumptions (DLP)
- ✓ BN254 curve parameters

**NOT Trusted**:
- ✗ Sequencer (all operations verified on-chain)
- ✗ Frontend (client-side proof generation possible)
- ✗ Network infrastructure (encrypted communications)

---

## 📊 Performance Characteristics

### Latency Breakdown

```
Total Transaction Time: ~4.5 seconds

├─ Frontend: Commitment generation       ~100ms  (  2%)
├─ Sequencer: Proof generation          ~2100ms ( 47%)
├─ Network: L1 propagation               ~800ms ( 18%)
├─ Blockchain: Consensus + verification ~1200ms ( 27%)
└─ Indexer: Event processing             ~300ms (  6%)
```

### Scalability

**Current (v1.0)**:
- Sequential proof generation: ~0.5 TPS
- Parallel (8 cores): ~4 TPS
- Batch verification: ~666 TPS (on-chain)

**Roadmap Goals**:
- GPU acceleration: ~50 TPS (Q2 2026)
- Recursive proofs: ~200 TPS (Q4 2026)
- Multi-sequencer: ~1000 TPS (2027)

---

## 🛠️ Technology Stack

### Layer 1 (Blockchain)
- **Platform**: Sui Network
- **Language**: Move
- **Consensus**: Narwhal + Bullshark
- **Finality**: ~3-4 seconds

### Layer 2 (Sequencer)
- **Language**: Rust 1.70+
- **Async Runtime**: Tokio
- **Parallelism**: Rayon
- **Cryptography**: arkworks, bellman

### Zero-Knowledge Proofs
- **Scheme**: Groth16
- **Curve**: BN254 (alt_bn128)
- **Hash**: Poseidon
- **Field**: Fr (scalar field)

### Development Tools
- **Build**: Cargo, Sui CLI
- **Testing**: cargo test, sui move test
- **Linting**: clippy, rustfmt
- **CI/CD**: GitHub Actions

---

## 🔮 Future Architecture Enhancements

### Phase 2: Federated Sequencers (Q3 2026)
```
Multiple Sequencers → Batch Aggregation → L1 Settlement
```

### Phase 3: Recursive Proofs (Q4 2026)
```
Proof₁ + Proof₂ + ... + ProofN → Single Aggregated Proof → L1
```

### Phase 4: Cross-Chain Bridges (2027)
```
Sui ↔ Ethereum ↔ Solana (via ZK proofs)
```

---

## 📚 Related Documentation

- [Whitepaper](../WHITEPAPER.md) - Full technical specification
- [Contributing Guide](../CONTRIBUTING.md) - How to contribute
- [API Reference](./API.md) - Developer documentation
- [Deployment Guide](./DEPLOYMENT.md) - Production setup

---

**Designed & Maintained by magic**  
*The Golden Triangle Method: Where Mathematics, Logic, and Performance Create Uncompromising Privacy*
