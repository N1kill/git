---
tags: [ghost-unit, reference]
---

# Glossary

Related: [[00 - Overview]]

## Nonce
A random value used exactly once. Its job is to make an old message unusable — if a system remembers every nonce it's already seen, replaying an old message (which includes an old nonce) always gets caught.

## Digital signature (ECDSA)
Proves a specific piece of data was approved by whoever holds a specific private key, without revealing that key. Anyone can check the signature using the matching public key. Changing even one character of the signed data invalidates the signature. Ethereum's native signature scheme.

## Perceptual hash (pHash)
Unlike a cryptographic hash (changes completely if even one pixel changes), a perceptual hash stays similar for visually similar images. Answers "does this look basically the same," not "is this byte-for-byte identical."

## Smart contract
A program living on a blockchain. Once deployed, code can't be secretly changed, and anyone can see exactly what rules it enforces. In Ghost Unit: the contract that refuses to accept an already-spent nonce.

## Replay attack
Any attack where a valid message from the past is captured and resubmitted later to trigger the same effect again — attacker never needs to understand or break the underlying cryptography.

## Liveness detection
A check designed to confirm something is happening in real time in front of a camera right now — as opposed to a photo, a screen replaying a recording, or any other pre-made stand-in.

## Threat model
A precise, written-down statement of exactly what an attacker can and can't do. Security claims only mean something once the threat model is explicit — "secure" alone is meaningless without saying secure *against what*.

## Testnet
A copy of a blockchain network used for testing, where the "money" involved is fake and free to obtain (faucets), so contracts can be deployed and broken without real financial risk. Using: **Polygon Amoy testnet**.

## Solidity (.sol files)
Programming language for writing smart contracts on Ethereum-compatible chains. Once deployed, generally can't be changed. Every operation costs gas. Everything stored/executed is publicly readable.

## Gas
Fee paid (in the network's cryptocurrency) for executing operations on a blockchain, because code runs on thousands of nodes worldwide, not just one machine.

## Hardhat
Development framework for writing, compiling, testing, and deploying smart contracts. Using v3 (ESM, TypeScript-first, Ignition for deployment — different conventions from Hardhat 2 tutorials).

## Ignition
Hardhat 3's deployment system (replaced the old "scripts/deploy.js" pattern from Hardhat 2).

## ethers.js
JavaScript/TypeScript library for interacting with Ethereum-compatible blockchains from code — calling contract functions, sending signed transactions, listening for events.
