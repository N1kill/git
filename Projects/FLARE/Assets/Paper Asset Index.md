---
tags: [flare, assets]
---

# Paper Asset Index

Part of [[FLARE - Project Hub|FLARE]]. Index of non-markdown assets related to FLARE — the vault stores text/Mermaid only, so binary files (PDFs, images) are tracked here by reference rather than embedded.

## Source Paper
- **File**: `main.pdf` — "Multi-Agent AI Platform for Automated Digital Forensic Investigation"
- **Authors**: Aluru Unnathi, Ande Tanmaya, Bhonagiri Sanjana, Afrah Sultana, Koyya Nikhil Sai Reddy — Dept. of CSE, Keshav Memorial Institute of Technology, Hyderabad
- **Not stored in this vault** — the vault's write tools are text/JSON-only (5MB text limit, no binary upload). Keep the PDF alongside your project code repo or in a dedicated `attachments`/PDF folder synced separately, and update this note with its path if that changes.
- Diagram content extracted from the PDF is recreated as Mermaid in this vault: [[System Architecture Diagram]] (Fig. 1) and [[Hierarchical RAG Pipeline Diagram]] (Fig. 2).
- Tables from the PDF are transcribed inline into the relevant architecture/decision notes rather than duplicated as images — see [[Go Disk Acquisition Agent]] (Table IV), [[RAG Ingestion Pipeline]] (Tables V, VI), [[Multi-Agent System Design]] (Table II).

## Recommendation
If you want the actual PDF browsable from inside Obsidian (not just referenced), the standard pattern is an `attachments/` folder at the vault root with binary files added directly through the Obsidian app (drag-and-drop) rather than through this MCP connection — then link to it here as `![[main.pdf]]`. Let me know if you'd like me to set up that folder structure.

## Related
- [[FLARE - Project Hub]]
- [[Research Paper - Overview]]
- [[Paper Review Notes]]
