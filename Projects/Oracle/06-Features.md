# Oracle — Features

What Oracle actually offers, end to end.

## Core prediction engine
- **Probabilistic price-move forecasts** for 7-day, 30-day, and 90-day horizons — not a single "buy/sell" call, but a calibrated probability (e.g. "68% chance of a >2% upward move in 30 days")
- Coverage across **thousands of tickers**, spanning **multiple global markets** (not just US — UK stocks explicitly handled, pence/pound unit bug fixed)
- **2% directional threshold** used to define what counts as a meaningful move, keeping labels meaningful rather than noise
- **Calibrated confidence**, not raw model confidence — Platt + Isotonic calibration means a "70%" actually behaves like 70% historically, not an overconfident raw score

## Explainability
- **Narrator agent** — turns SHAP feature-importance values into plain-English explanations of *why* the model thinks what it thinks for a given ticker/prediction
- Gives users a reason, not just a number

## Market monitoring & alerts
- **Watchdog agent** — monitors Google News RSS in real time, uses an LLM (Groq LLaMA 8b) to detect when breaking news *contradicts* an active prediction, so stale signals get flagged fast
- **Telegram bot with 9 commands** — lets users query predictions, manage watchlists, and receive alerts directly in Telegram
- **Live signals page** (in progress) — Supabase realtime subscriptions push new/updated predictions to the frontend without a page refresh

## Portfolio tools
- **Portfolio Risk scorer agent** — uses Groq LLaMA 70b to assess risk across a user's holdings/watchlist, not just single-ticker predictions

## Discovery
- **Screener agent** — surfaces tickers matching criteria (e.g. high-probability movers) rather than requiring the user to already know what to look up

## Planned / in progress
- **Chat/RAG agent** — conversational interface for querying Oracle's data and reasoning (not yet built)
- **Live signals page** — architecture designed, implementation in progress

## Self-maintaining system
- **Automated prediction verification** — checks real market outcomes against what was predicted, feeding accuracy tracking
- **Automated retraining pipeline** — verified outcomes accumulate and periodically retrain the model, so Oracle is designed to improve itself over time rather than go stale
- This is the current focus area — verification data has piled up and a retrain is overdue (see [[04-Open-Issues]])

## User-facing surfaces
| Surface | Purpose |
|---|---|
| Web frontend (Next.js) | Main dashboard, glassmorphism UI, live signals |
| Telegram bot | Mobile-first access, alerts, 9 commands |
| (Planned) Chat/RAG | Natural-language Q&A over predictions |

## Design philosophy behind the features
Every feature reinforces the core motto — *"Not a prediction. A probability."* Nothing in Oracle is presented as a certainty: predictions are probabilistic, explanations show *why*, alerts flag when the world has changed, and the system checks its own track record and retrains rather than assuming it's still right.
