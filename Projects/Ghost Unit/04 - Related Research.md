---
tags: [ghost-unit, research, papers]
---

# Related Research

Related: [[00 - Overview]] · [[01 - The Attack (Clone-and-Replay)]] · [[02 - The Defense (Nonce + Liveness)]]

## Closest genre precedent
**"Exposing Vulnerabilities in Counterfeit Prevention Systems Utilizing Physically Unclonable Surface Features"**
arXiv:2512.09150, Dec 2025 — Nakra, Wu, Wong et al.
Designed digital forgery + physical DoS attacks against paper-PUF authentication. Proof that the "attack-on-provenance-authentication" paper format is actively being published right now. Ghost Unit attacks the blockchain-anchored digital layer (proof replay) — complementary, distinct attack surface from their physical-layer attacks.

## Structural gap validation
**"SoK: Fighting Counterfeits with Cyber-Physical Synergy Based on Physically-Unclonable Identifiers of Paper Surface"**
arXiv:2408.02221, Aug 2024 — Nakra, Wu, Wong
Systematizes the field; notes prior work improves unclonable identifiers OR secures digital records, never both cohesively. Ghost Unit targets the exact seam between a valid identifier and a fresh verification event.

## Nearest attempted fix — KEY comparison baseline
**"A Dual-QR Blockchain-Based Authentication Mechanism for Agricultural Anti-Counterfeiting"**
Frontiers in Blockchain, 2026
Public traceability QR + concealed single-use private QR; records only first successful verification on-chain. Fixes replay-of-the-identifier but NOT replay-of-the-verification-event (see [[01 - The Attack (Clone-and-Replay)]] for why this matters). This is the paper Ghost Unit directly improves on.

## Methodological cousin — freshness mechanism
**"A Lightweight QR-assisted Zero-knowledge Identification Protocol For Secure Authentication"**
arXiv:2605.16912, 2026
Timestamp-validity-window + Fiat–Shamir non-interactive ZK transform, rejects proofs outside tolerance window. Good citation for justifying the nonce/freshness-window design choice.

## Threat-model citation
**"zQR: A Verifiable QR-Driven zkSNARK Proof Verification Framework for Mobile Platforms"**
arXiv:2606.27092, 2026
Names QR payload injection + transaction-replaying as threat categories. Useful for framing "replay" as a recognized threat class, not invented for this proposal.

## Cross-domain replay precedent
**"QLink: Quantum-Safe Bridge Architecture for Blockchain Interoperability"**
arXiv:2512.18488, 2025
Documents proof forgery/replay against cross-chain bridges, cites Wormhole 2022 exploit. Evidence that replay-of-valid-proofs is a recognized, high-stakes pattern elsewhere in blockchain systems.

## Defense-pattern precedent (different domain)
**"InterPUF: Distributed Authentication via PUFs and Multi-party Computation for Reconfigurable Interposers"**
arXiv:2601.11368, 2026
Defends against replay via per-boot salts + session nonces binding authentication transcripts. Hardware-authentication analog of Ghost Unit's nonce-binding defense.

## Baseline design reference
**"Blockchain-Enabled Product Tokenization and CNN-Based Counterfeit Detection"**
Macaw Int'l Journal of Advanced Research in CSE, 2025
ERC-721 tokenization + QR verification, 94.2% CNN accuracy, 95.3% true-verification rate, 1.42s avg latency on Goerli testnet. Blueprint for what the naive baseline (the thing to attack) should look like.

## Suggested reading order for citations
Septillion (motivation, industry) → dual-QR + zQR + zk-QR (recognized threat + closest fix) → SoK + PUF attack paper (genre precedent) → QLink + InterPUF (cross-domain patterns) → Macaw (baseline design) → Ghost Unit, framed as filling the specific event-freshness gap.
