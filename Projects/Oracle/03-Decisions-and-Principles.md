# Oracle — Key Decisions & Principles

## Calibration over raw accuracy
Oracle explicitly frames outputs as probabilities, not predictions. This philosophical distinction shapes architecture decisions — calibration steps, honest validation, directional correctness checks all flow from it.

## XGBoost over LSTM
Reasoned selection, not default:
- Better fit for tabular data
- SHAP interpretability
- Faster training
- Better suited to dataset size constraints

## Data bugs are expensive
Several production accuracy issues traced back to subtle data handling errors (unit mismatches, direction-blindness), not model architecture. Lesson: audit data pipeline correctness before assuming the model needs work.

## Caching layers are distinct
- KV/prompt caching = compute layer
- Response caching = application layer
- These solve different problems — don't conflate them
- Live price data should not be aggressively cached
- Model predictions suit a realtime-invalidation pattern

## Agentic architecture maps naturally onto Oracle
Multi-agent orchestrator pattern (Watchdog, Signal, Portfolio Risk sub-agents), connected via MCP servers to real systems, fits Oracle's modularity well.

## Working style / approach
- Builds full-stack end-to-end rather than layer by layer — ML, backend, frontend, agents, ops all in motion simultaneously
- Asks foundational conceptual questions alongside implementation questions (why XGBoost, how caching works, what agentic AI is) — values understanding the reasoning, not just the code
- Uses visual/diagrammatic explanations productively (SVG architecture diagrams)
- Iterates on model versions with explicit version tracking and honest evaluation of generalization
