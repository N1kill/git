# Oracle 

**Motto:** *"Not a prediction. A probability."*
**Tagline:** *"Market speaks, we translate."*

AI-powered stock probability engine predicting calibrated probabilities (not deterministic predictions) for 7-day, 30-day, and 90-day price moves across thousands of tickers spanning multiple global markets.

Full-stack, production-grade product: ML training, backend API, frontend UI, autonomous agents, Telegram bot, automated retraining pipelines.

**Repo:** https://github.com/N1kill/Stock

## Notes in this folder
- [[01-Architecture]] — system design across all layers
- [[02-Model-Status]] — ML versions, accuracy, calibration
- [[03-Decisions-and-Principles]] — key reasoning and learnings
- [[04-Open-Issues]] — bugs, debugging, on-the-horizon work
- [[05-Daily-Log]] — running log of sessions/progress
- [[06-Features]] — what Oracle actually offers, feature by feature

## Current phase
Critical maintenance and accuracy-recovery phase. Substantially built across every layer, but production verification data shows 7d/30d accuracy well below training estimates. Immediate retraining recommended.
