---
tags: [ghost-unit, roadmap]
---

# Build Roadmap (10–12 weeks)

Related: [[00 - Overview]] · [[05 - Tech Stack and Architecture]] · [[07 - Progress Log]]

## Week 1–2 — Build the naive baseline
Static QR + on-chain hash system, matching published patterns (Macaw's ERC-721 tokenization or simple SHA-256 + Solidity registry). Deploy to Polygon Amoy. Must be a fair, realistic target — not a strawman.

## Week 3 — Execute and formalize the attack
Build scrape-and-replay harness. Run N replay attempts, measure success rate (expect ~100%). Write the formal attack model: adversary capability, what's captured, what's replayed, what the chain sees.

## Week 4 — Design the nonce + signature protocol
Design on paper first: payload structure, signature scheme (ECDSA), nonce lifetime, exact security property being claimed.

## Week 5–6 — Implement the hardened system
Nonce-issuing FastAPI service + Redis TTL store. Extend smart contract with nonce-burn checks + ECDSA verification. Re-run the exact Week 3 attack — should now fail.

## Week 7 — Add the liveness layer
Video capture instead of photo. OpenCV optical-flow check for natural motion. Stops the second-order screen-replay attack.

## Week 8 — Attempt to break your own defense
Screen-replay attack against hardened system. Timing attacks against nonce window. Signature-splicing attempts. Document what works, what doesn't, why.

## Week 9 — Full evaluation sweep
Attack success rate (baseline vs hardened), liveness false-reject rate at different thresholds, latency + gas-cost overhead, nonce-window-length vs security tradeoff curve.

## Week 10–12 — Write-up
Position against dual-QR paper, SoK, PUF attack paper, zkSNARK-QR/Fiat-Shamir papers. State threat model boundary clearly.

---

## Local dev sequencing (granular, current phase)
1. ✅ Node/npm verified (v22.12.0 / 10.9.0)
2. ✅ Hardhat installed + project scaffolded (TS + Mocha + ethers.js)
3. ✅ Starter compile confirmed working (solc 0.8.28)
4. ⏳ Clear starter files (Lock.sol etc.) — in progress, filenames didn't match, need to check actual `dir` output
5. ⬜ Write `ProductRegistry.sol` — baseline vulnerable contract
6. ⬜ Write deployment Ignition module
7. ⬜ Deploy to local Hardhat network, test read/write
8. ⬜ Deploy to Polygon Amoy testnet
9. ⬜ Build attack harness script
