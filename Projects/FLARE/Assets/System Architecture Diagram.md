---
tags: [flare, assets, diagram]
---

# System Architecture Diagram (Paper Fig. 1)

Part of [[FLARE - Project Hub|FLARE]]. Recreation of the paper's Figure 1 — overall system architecture — as Mermaid, since the vault can't embed the PDF's original image directly. See [[Paper Asset Index]] for where the source PDF lives on disk.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor':'#ffffff', 'primaryBorderColor':'#000000', 'background':'#ffffff', 'mainBkg':'#ffffff', 'clusterBkg':'#ffffff', 'clusterBorder':'#000000', 'fontSize':'15px'}}}%%
graph TB
    DISK["Disk Images"]
    LOGS["Log Files"]

    subgraph diskpipe["Disk Image Pipeline"]
        GOENG["Go imaging engine (cross-platform)"]
        TSKD["TSK in Docker container"]
        OV["Overview"]
        PART["Partition"]
        CARVE["Data carve"]
        META["Metadata"]
        TIME["Timeline"]
        GOENG --> TSKD
        TSKD --> OV
        TSKD --> PART
        TSKD --> CARVE
        TSKD --> META
        TSKD --> TIME
    end

    subgraph logpipe["Log File Pipeline"]
        CHUNK["Hierarchical Chunking (L0-L3)"]
        EMBED["BAAI / bge-small-en embedding"]
        VIZ["Visual Analytics — 4 parallel agents"]
        CHUNK --> EMBED
        EMBED --> VIZ
    end

    DISK --> diskpipe
    LOGS --> logpipe

    diskpipe --> STORE["Evidence Storage — Supabase + PostgreSQL + pgvector"]
    logpipe --> STORE

    STORE --> ORCH["Orchestrator Agent"]
    STORE --> REPORT["Report Generation Engine — Judiciary + Executive + Formatting"]

    ORCH --> SEM["Semantic — CrossEncoder re-ranker"]
    ORCH --> STAT["Statistical — Code gen + sandbox"]

    SEM --> VALID["Validator Agent — Hallucination Mitigation"]
    STAT --> VALID
```

## Related
- [[Hierarchical RAG Pipeline Diagram]] — paper Fig. 2, the RAG-specific detail view
- [[Multi-Agent System Design]]
- [[FLARE - Project Hub]]
