# Oracle — Open Issues & On the Horizon

## Immediate priorities
- [ ] Retrain models using newly accumulated verified data (updated hyperparameters)
- [ ] Complete live signals page (Supabase realtime)
- [ ] Create tailored deployment playbooks for Oracle
- [ ] Set up ongoing model monitoring / accuracy tracking post-retrain

## Resolved
- [x] Telegram alert-firing issue in GitHub Actions pipeline — **fixed**

## Tools & stack reference
- ML: XGBoost, SHAP, Platt/Isotonic calibration, walk-forward validation
- Backend: FastAPI, Python
- Frontend: Next.js 14, TypeScript, TSX
- DB: Supabase (PostgreSQL + realtime)
- Agents/LLMs: Groq (LLaMA 8b, 70b), Google News RSS
- CI/CD: GitHub Actions
- Deployment: Railway, Vercel, Docker
- Bot: Telegram Bot API
- Repo: github.com/N1kill/Stock
