---
tags: [ghost-unit, log]
---

# Progress Log

Related: [[00 - Overview]] · [[06 - Build Roadmap]]

## 2026-07 — Local environment setup
- OS: Windows
- Node v22.12.0, npm 10.9.0 confirmed
- Project folder: `ghost-unit/` (nested under a deep local path on D: drive)
- `npm init -y` → package.json created
- Hardhat installed via `npm install --save-dev hardhat`
- `npx hardhat --version` → confirmed v3.12.0 working
- Ran `npx hardhat --init` (note: Hardhat 3 uses `--init` flag, not `init` subcommand — `npx hardhat init` throws HHE3 "no config file found")
- Project type selected: **TypeScript Hardhat project using Mocha and Ethers.js** (chosen over Node Test Runner + Viem option and the minimal option — ethers.js matches the planned stack, Mocha is more beginner-documented)
- Converted package.json to ESM (accepted prompt)
- Dependencies installed: 211 packages, several deprecation warnings (glob, inflight — safe to ignore, sub-dependency noise), 14 vulnerabilities flagged by npm audit (low/moderate, not blocking, NOT running `audit fix --force` — risk of breaking Hardhat's expected versions)
- `npx hardhat compile` → **success**, compiled 2 Solidity files with solc 0.8.28 (evm target: cancun), auto-downloaded compiler

## Known open issue
Attempted to delete starter files:
```
del contracts\Lock.sol
del test\Lock.ts
del ignition\modules\Lock.ts
```
All three returned "Could Not Find" — filenames in this Hardhat 3 Mocha+Ethers template likely differ from the Hardhat 2-era `Lock.sol` convention. Next step: run `dir contracts`, `dir test`, `dir ignition\modules` to see actual filenames before deleting anything.

## Next up
- Resolve starter file naming, delete correctly
- Write `ProductRegistry.sol` (baseline contract) — see [[05 - Tech Stack and Architecture]]
