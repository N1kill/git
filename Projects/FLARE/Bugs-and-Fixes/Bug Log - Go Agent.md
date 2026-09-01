---
tags: [flare, bugs, go]
component: disk-acquisition
---

# Bug Log — Go Disk Acquisition Agent

Part of [[FLARE - Project Hub|FLARE]]. Five critical stubs identified and fixed in the [[Go Disk Acquisition Agent]].

## 1. Fake Drive Enumerator
- **Problem**: drive enumeration was stubbed/fake rather than querying the actual system.
- **Fix**: replaced with real enumeration logic, confirmed against actual NVMe hardware with correct partition data returned.

## 2. Simulated Acquisition
- **Problem**: the acquisition step simulated imaging instead of performing real WinAPI-based disk acquisition.
- **Fix**: implemented real acquisition via WinAPI calls.

## 3. Missing `sync.WaitGroup`
- **Problem**: concurrent goroutines were not properly synchronized, risking premature exit or race conditions.
- **Fix**: added proper `sync.WaitGroup` usage to ensure all goroutines complete before proceeding.

## 4. Empty Hashes
- **Problem**: integrity hashing step was returning empty/placeholder hashes rather than real computed values — a critical issue for evidentiary chain-of-custody.
- **Fix**: implemented real hash computation over acquired data.

## 5. Always-False Admin Check
- **Problem**: the privilege check always evaluated to `false`, regardless of actual process privilege level.
- **Fix**: implemented a real elevated-privilege check.

## Verification

Confirmed working on Windows: produces real output against an actual NVMe drive, with correct partition enumeration.

## Related

- [[Go Disk Acquisition Agent]]
- [[Bug Log - RAG Pipeline]]
