---
tags: [ghost-unit, project-hub]
---

# Ghost Unit — Overview

Security research project: formalize a **clone-and-replay attack** against blockchain-based physical product provenance systems, then design and prove a **challenge-response liveness defense** that closes it.

## The one-line pitch
Every blockchain product-authentication system assumes that a valid cryptographic proof means the physical scan just happened, on the real object. Nothing enforces that. A proof that verified once verifies forever, anywhere. Ghost Unit demonstrates this attack, then fixes it.

## Core notes
- [[01 - The Attack (Clone-and-Replay)]]
- [[02 - The Defense (Nonce + Liveness)]]
- [[03 - Glossary]]
- [[04 - Related Research]]
- [[05 - Tech Stack and Architecture]]
- [[06 - Build Roadmap]]
- [[07 - Progress Log]]

## Target outcome
- Working baseline (vulnerable) system
- Demonstrated attack with measured success rate
- Hardened defense system
- Demonstrated defense closing the attack, with measured tradeoffs (latency, false-reject rate)
- Write-up targeting workshop/mid-tier venue (IEEE ICBC / ACM AFT tier)

## Status
Currently: local Hardhat + TypeScript + Mocha + ethers.js project scaffolded on Windows. See [[07 - Progress Log]] for exact state.
