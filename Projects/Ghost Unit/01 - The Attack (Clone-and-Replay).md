---
tags: [ghost-unit, theory, attack]
---

# The Attack — Clone-and-Replay

Related: [[00 - Overview]] · [[02 - The Defense (Nonce + Liveness)]] · [[03 - Glossary]]

## The core assumption every naive system makes
If a cryptographic proof verifies, the physical scan that produced it must have just happened, on the real object. **This is never actually enforced by static QR/hash/NFC systems.**

## Why blockchain alone can't catch this
Analogy: a notary stamp proves a document existed unchanged since a date — it does NOT prove the document told the truth. A blockchain product record works the same way: it proves "this hash was recorded on this date," not "the object this hash describes is genuine right now."

## The attack sequence
1. Attacker buys **one genuine unit** — only physical interaction needed
2. Scans it once, extracts the **valid proof** (QR payload / NFC bytes / signed hash response)
3. Manufactures unlimited fakes — physical quality doesn't matter, never inspected again
4. Stamps the **identical stolen proof** onto every fake
5. Every fake passes verification — the crypto is 100% valid, because it IS the same valid proof, just copied

## Why this works
The attacker never touches the model, the contract, or the cryptography. They only **replay**, never **break**, anything. Success rate against a naive system → expected ~100%.

## Why this isn't hypothetical
- Dec 2025 arXiv paper (2512.09150) independently found digital forgery + physical DoS attacks against paper-PUF authentication — even physically-unclonable-identifier systems have unaddressed replay-shaped holes
- 2026 Frontiers dual-QR paper explicitly names "unit-level authenticity enforcement" as a persistent unresolved challenge

## Why the closest existing fix (dual-QR) isn't enough
Dual-QR marks the **identifier** as used-once (first scan wins, on-chain). Problems:
- Breaks legitimate re-verification (customer checking twice, customs re-scanning)
- Does nothing if attacker's replay reaches the chain **before** the genuine unit is ever scanned — genuine buyer gets locked out instead of the fake
- Freshness needs to be tied to the **verification event**, not a flag on the identifier itself — this is the gap Ghost Unit's nonce design closes

See [[02 - The Defense (Nonce + Liveness)]] for the fix.
