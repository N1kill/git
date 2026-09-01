---
tags: [hackathon, timeline]
status: planning
---

# Build Timeline

Deadline: **Sep 14, 2026**. Effort split ~65% [[Household-to-Habitat]] / ~35% [[Relocation Copilot]].

← back to [[00 Index]]

## Week-by-week

### Week 1 — Shared foundation + eligibility data start
- [ ] Stand up shared Strands + Bedrock boilerplate (model client, base orchestrator)
- [ ] Set up shared Streamlit shell
- [ ] **Start** the Household-to-Habitat eligibility-rules knowledge base (biggest unplanned-time risk — do not defer this)
- [ ] Pick the one city for Relocation Copilot's scope

### Week 2 — Household-to-Habitat core
- [ ] Intake + Triage agent
- [ ] Eligibility agent wired to the rules KB
- [ ] Graph branch logic (subsidized vs. private-pay)

### Week 3 — Household-to-Habitat branches + Relocation Copilot start
- [ ] Contractor matcher + mock directory
- [ ] Nonprofit application drafting agent
- [ ] Integrate official human-in-the-loop approval sample
- [ ] Relocation Copilot: intake + personal-move sub-agent

### Week 4 — Relocation Copilot core + integration
- [ ] Relocation Copilot: career-transition sub-agent
- [ ] Agents-as-Tools orchestration + timeline merge
- [ ] Memory-agent integration for session persistence
- [ ] Household-to-Habitat: scheduling + status-tracker agents

### Week 5 — Deployment + polish
- [ ] Household-to-Habitat → Bedrock AgentCore Runtime deployment
- [ ] Relocation Copilot → Lambda/Fargate deployment
- [ ] Reskin shared Streamlit shell per project
- [ ] End-to-end test both demo scenarios (subsidized + private-pay paths; job-searching relocation scenario)

### Week 6 — Submission prep
- [ ] Architecture diagrams for both (Devpost submission requirement)
- [ ] Demo videos (≤5 min each)
- [ ] READMEs + MIT/Apache license + public GitHub repos
- [ ] AWS Builder ID confirmed
- [ ] Optional: build-story post on builder.aws.com (bonus points)
- [ ] Confirm Devpost multiple-submission rule before final submit
- [ ] Submit both

## Risk watch

| Risk | Project | Mitigation |
|---|---|---|
| Eligibility KB eats unplanned time | Household-to-Habitat | Started Week 1, not deferred |
| Demo reads as generic/toy | Relocation Copilot | Hard-scoped to one city from the start |
| Two builds cannibalize each other's time | Both | 65/35 split is deliberate, not equal |
| Contractor vetting scope creep | Household-to-Habitat | Explicitly out of scope — pre-vetted mock directory only |
