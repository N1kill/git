---
tags: [ghost-unit, theory, defense]
---

# The Defense — Nonce Binding + Liveness Detection

Related: [[00 - Overview]] · [[01 - The Attack (Clone-and-Replay)]] · [[03 - Glossary]]

## The core idea
Force every verification to require something that:
- Expires in seconds
- Can only be produced by physically having the real object **right now**

= a **nonce** + a **live capture** (video, not static photo).

## Mechanism, step by step (genuine check)
1. Verifier app requests a challenge from the server
2. Server generates a **fresh, single-use nonce** (~5–10 sec validity, stored in Redis/short-TTL store)
3. Nonce is shown on the verifier's own screen; a short video is captured showing product + nonce + natural motion/parallax
4. Video is analyzed: motion-detection check (rules out static loops/screen replay) + perceptual hash computed from frames
5. Payload bundled: `{product_id, nonce, pHash, timestamp}` → signed with verifier service's private key (ECDSA)
6. Smart contract checks: (a) signature valid, (b) nonce never used before → accepts, marks nonce spent

## Mechanism, step by step (replayed/stolen proof)
1. Attacker has a full valid payload from an earlier real scan (signature, nonce, hash — all mathematically valid)
2. Resubmits it against a fake product
3. Signature check passes (nothing tampered)
4. **Nonce check fails** — already marked spent from the first legitimate use
5. Transaction reverts — not because the math is wrong, but because freshness fails

## Why a nonce beats "one-time-use identifier" (the dual-QR approach)
Binding freshness to the **verification event** (nonce) rather than the **identifier** (dual-QR) means order-of-arrival never matters. Every verification — genuine or fake — needs its own fresh proof, regardless of who scans first.

## Why liveness needs a CV check too, not just a nonce
A nonce alone stops replaying an *old* payload. Doesn't stop: attacker holds a real unit up to a second screen looping a pre-recorded "live" video with the nonce composited in. This is the **second-order attack** — liveness detection (optical flow / motion parallax / natural hand-shake micro-movement) is what closes it.

## Honest limitation (say this explicitly in any write-up)
This defense stops **reusing an old proof**. It does NOT stop someone building a genuinely convincing physical fake and getting a fresh, legitimate verification on that fake. That's a separate, harder problem (AI detection accuracy, not proof freshness). Ghost Unit closes the replay door, not every door.

## Security claim (to formalize properly later)
A replayed proof is rejected with probability ≥ 1 − negligible, under a stated adversary model. Need to define the adversary model precisely before claiming this in the paper — see [[06 - Build Roadmap]] Week 4.
