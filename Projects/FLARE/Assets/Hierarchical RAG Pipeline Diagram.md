---
tags: [flare, assets, diagram]
---

# Hierarchical RAG Pipeline Diagram (Paper Fig. 2)

Part of [[FLARE - Project Hub|FLARE]]. Recreation of the paper's Figure 2 — the hybrid retrieval + re-ranking flow inside the [[RAG Ingestion Pipeline]].

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor':'#ffffff', 'primaryBorderColor':'#000000', 'background':'#ffffff', 'mainBkg':'#ffffff', 'clusterBkg':'#ffffff', 'clusterBorder':'#000000', 'fontSize':'15px'}}}%%
graph TB
    ORCH["Orchestrator"]
    ORCH --> SEM["Semantic"]
    ORCH --> STORE["Storage — Supabase"]
    ORCH --> STAT["Statistic"]

    SEM --> EMBEDQ["Embed query"]
    EMBEDQ --> PGV["pgvector store — Hybrid: semantic + SQL filter"]
    STORE --> PGV
    STORE --> DF["Dataframe"]
    STAT --> QUERY["Query"]
    DF --> QUERY
    QUERY --> GENPY["Generates python code"]

    PGV --> TOP50["Top-50 candidates — Vector similarity recall"]
    TOP50 --> CE["CrossEncoder re-rank — ms-marco, top-10"]
    CE --> LLMGEN["LLM Answer Generation"]
```

## Key numbers attached to this flow (paper Table VI)

| Stage | Avg. Latency |
|---|---|
| Hybrid pgvector query | 120ms |
| Semantic similarity scoring | 45ms |
| Keyword match (regex) | 15ms |
| Cross-Encoder re-ranking | 340ms |
| Context window population | 10ms |
| **Total** | **530ms** |

## Related
- [[System Architecture Diagram]] — paper Fig. 1, the full-system view
- [[RAG Ingestion Pipeline]]
- [[FLARE - Project Hub]]
