---
tags: [ghost-unit, tech-stack, architecture]
---

# Tech Stack and Architecture

Related: [[00 - Overview]] · [[06 - Build Roadmap]]

## Two systems to build
1. **Baseline (vulnerable)** — realistic naive system, matches published literature, this is what gets attacked
2. **Hardened (defended)** — same pipeline + nonce/liveness layer added

## Baseline stack
- **Chain:** Solidity, Hardhat, Polygon Amoy testnet, ethers.js
- **Capture:** Static QR (qrcode.js), simulated NFC payload, SHA-256/imagehash pHash
- **Attack harness:** Python scripts to capture + replay payloads; Selenium/Playwright for automated re-scan simulation

## Defense additions
- **Nonce layer:** FastAPI nonce-issuing service, Redis (short-TTL nonce store), HMAC/ECDSA payload signing
- **Liveness check:** OpenCV optical flow (motion/parallax detection), MediaPipe (hand-shake micro-motion), optional lightweight CNN anti-spoof classifier
- **Smart contract additions:** nonce-used mapping (`mapping(bytes32 => bool)`), ECDSA signature verification (OpenZeppelin ECDSA lib)

## Supporting
- **Mobile/capture app:** React Native or plain mobile-web camera capture page, short video not photo
- **Evaluation/stats:** Python (numpy/pandas), matplotlib for tradeoff curves

## Why Polygon testnet, not permissioned chain
Contribution is about the capture/verification layer, not chain governance. Public EVM testnet = fastest path to reproducible, publicly-inspectable demo. Discuss Hyperledger/permissioned deployment as future work (pharma/regulated industries) in conclusion only.

## Architecture flow (baseline → attack)
Genuine unit → manufacturer mints QR/hash on-chain (static, reusable) → attacker scans once, extracts full valid payload → stamped onto N fakes → every fake verifies as genuine (100% pass) → chain never sees the difference

## Architecture flow (hardened → defended)
Verifier requests challenge → server issues fresh nonce (~5-10s expiry) → live capture required (video, motion/parallax checked) → signed nonce-bound payload → on-chain accept/revert, nonce burned after use

## Current local environment (Windows)
- Node.js v22.12.0, npm 10.9.0
- Hardhat v3.12.0 (Ignition deployment system, ESM + TypeScript)
- Project type selected: TypeScript Hardhat project using Mocha and Ethers.js
- See [[07 - Progress Log]] for exact setup state
