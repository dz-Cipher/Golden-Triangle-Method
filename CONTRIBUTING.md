# Contributing to Golden Triangle Method

**Designed & Maintained by magic**

Thank you for your interest in contributing to the Golden Triangle Method! This document provides guidelines for contributing to this open-source privacy framework.

---

## 🌟 Ways to Contribute

### 1. **MATH (Cryptography) Contributions**
- Optimize zkSNARK circuits
- Implement new proof systems
- Conduct security audits
- Research quantum-resistant upgrades

**Skill Level**: Advanced  
**Requirements**: Strong cryptography background, familiarity with zero-knowledge proofs

### 2. **MOVE (Smart Contracts) Contributions**
- Add new privacy pool features
- Optimize gas consumption
- Write formal verification specs
- Improve contract documentation

**Skill Level**: Intermediate  
**Requirements**: Move language experience, blockchain development

### 3. **RUST (Sequencer) Contributions**
- Improve proof generation performance
- Add GPU acceleration
- Implement parallel processing optimizations
- Enhance error handling

**Skill Level**: Intermediate  
**Requirements**: Rust proficiency, systems programming

### 4. **Documentation**
- Write tutorials and guides
- Translate documentation
- Create video explanations
- Improve API documentation

**Skill Level**: Beginner to Intermediate  
**Requirements**: Clear writing skills

### 5. **Testing & Quality Assurance**
- Write integration tests
- Conduct fuzzing campaigns
- Benchmark performance
- Report bugs

**Skill Level**: Intermediate  
**Requirements**: Testing frameworks, attention to detail

---

## 🔧 Development Setup

### Prerequisites
```bash
# Rust (1.70+)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Sui CLI
cargo install --locked --git https://github.com/MystenLabs/sui.git --branch testnet sui

# Node.js (for frontend)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
```

### Clone & Build
```bash
git clone https://github.com/dz-Cipher/Golden-Triangle-Method.git
cd Golden-Triangle-Method

# Build Rust sequencer
cargo build --release

# Build Move contracts
cd contracts
sui move build

# Run tests
cargo test
sui move test
```

---

## 📝 Submission Guidelines

### Pull Request Process

1. **Fork the repository**
   ```bash
   git clone https://github.com/YOUR_USERNAME/Golden-Triangle-Method.git
   cd Golden-Triangle-Method
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes**
   - Write clear, commented code
   - Follow existing code style
   - Add tests for new features
   - Update documentation

3. **Test thoroughly**
   ```bash
   cargo test
   cargo clippy
   cargo fmt --check
   sui move test
   ```

4. **Commit with clear messages**
   ```bash
   git commit -m "feat(math): optimize Groth16 proof generation"
   ```

   **Commit message format**:
   - `feat(scope)`: New feature
   - `fix(scope)`: Bug fix
   - `docs(scope)`: Documentation
   - `perf(scope)`: Performance improvement
   - `test(scope)`: Testing
   - `refactor(scope)`: Code refactoring

5. **Push and create PR**
   ```bash
   git push origin feature/your-feature-name
   ```

### Code Review Process

1. Automated tests must pass
2. Code review by core team
3. Security review (for cryptographic changes)
4. Community feedback period (major changes)
5. Approval and merge

---

## 🛡️ Security Contributions

### Reporting Vulnerabilities

**DO NOT** create public GitHub issues for security vulnerabilities.

Instead:
1. Email: security@[your-domain].com (replace with actual contact)
2. Include:
   - Vulnerability description
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

3. We will respond within 48 hours
4. Coordinated disclosure timeline: 90 days

### Security Bounties

Critical vulnerabilities may be eligible for rewards:
- **Critical**: $5,000 - $10,000
- **High**: $1,000 - $5,000
- **Medium**: $500 - $1,000
- **Low**: $100 - $500

---

## 📋 Code Standards

### Rust
```rust
// Use clear, descriptive names
fn generate_groth16_proof(transaction: &Transaction) -> Result<Proof, Error> {
  // Comprehensive error handling
  let witness = transaction.build_witness()
    .map_err(|e| Error::WitnessGeneration(e))?;
  
  // Document complex logic
  // Generate proof using Groth16 scheme with BN254 curve
  let proof = groth16::create_proof(witness, &PARAMS)?;
  
  Ok(proof)
}
```

### Move
```move
/// Brief description of function purpose
/// 
/// # Arguments
/// * `pool` - The privacy pool to deposit into
/// * `coin` - The coin to deposit
/// 
/// # Aborts
/// * If deposit amount is zero
/// * If commitment is invalid
public entry fun deposit(
  pool: &mut PrivacyPool,
  coin: Coin<SUI>,
  commitment: vector<u8>,
  ctx: &mut TxContext
) {
  // Implementation
}
```

---

## 🗺️ Roadmap Alignment

Prioritize contributions that align with the [project roadmap](../WHITEPAPER.md#104-roadmap):

**Q2 2026 Focus**:
- GPU-accelerated proof generation
- Mobile SDK development
- Mainnet preparation

**Q3 2026 Focus**:
- Federated sequencer network
- Recursive proof aggregation
- Multi-asset support

---

## 💬 Community

- **GitHub Discussions**: Ask questions, propose features
- **Discord**: Real-time collaboration (link TBD)
- **Twitter**: Follow [@GoldenTriangleMethod](https://twitter.com/...) for updates

---

## 📚 Additional Resources

- [Whitepaper](../WHITEPAPER.md)
- [Architecture Documentation](./ARCHITECTURE.md)
- [API Reference](./API.md)
- [Testing Guide](./TESTING.md)

---

## 🙏 Recognition

All contributors will be acknowledged in:
- `CONTRIBUTORS.md` file
- Release notes
- Project website

Significant contributions may result in:
- Author credit in academic papers
- Speaking opportunities at conferences
- Advisory board positions

---

## 📜 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Designed & Maintained by magic**  
*Building the future of privacy-preserving blockchain systems, together.*
