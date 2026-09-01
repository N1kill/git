---
tags: [hackathon, agents-for-humans, strands-sdk]
deadline: 2026-09-14
status: active
---

# Agents for Humans — Hackathon Index

AWS-sponsored Devpost hackathon. Build agents with the **Strands Agents SDK**. $40,000 prize pool across three tracks (Everyday / Professional / Good Neighbor) plus a $10,000 grand prize. Deadline **Sep 14, 2026**.

Strategy: build **two projects simultaneously**, split effort ~65/35, submit both (Devpost generally allows multiple submissions per entrant — confirm on the actual rules page before final submission).

## The two bets

| | [[Household-to-Habitat]] | [[Relocation Copilot]] |
|---|---|---|
| Effort split | ~65% (flagship) | ~35% (scoped down) |
| Tracks fused | Everyday × Professional × Good Neighbor | Everyday × Professional |
| Why | Only idea where all 3 tracks are load-bearing; least crowded track; no AWS sample overlaps it | Genuinely unclaimed pairing; different track exposure; different technical muscle (integrations vs. eligibility-reasoning) |
| Risk | Eligibility-rules complexity easy to oversimplify | Demo only convincing if scoped to one city |
| Deployment | Bedrock AgentCore Runtime (full) | Lambda/Fargate (lighter) |

## Shared infrastructure (build once, reuse across both)

- Streamlit UI shell (base on AWS's official `streamlit-template` sample)
- Human-in-the-loop interrupt/resume component (base on AWS's official `human-in-the-loop-approval-agent` sample)
- Memory-agent pattern for persisting user state across sessions (base on official `memory_agent` example)
- Bedrock (Claude) as model provider — shared client setup
- Base Strands orchestrator boilerplate

## Competitive landscape notes

- AWS's own sample repo (`github.com/strands-agents/samples`) has **zero** nonprofit/community/volunteer industry examples — confirms Good Neighbor is the structurally thin track
- No AWS sample or major existing product found automating the "contractor vs. subsidized-nonprofit" triage — Household-to-Habitat's core gap
- No product found fusing personal move-logistics with career-transition — Relocation Copilot's core gap
- Full research/critique of 9 single-track + 11 fused ideas lives outside this vault, in the two HTML blueprint docs from the Claude conversation — worth re-pulling into notes here if useful later

## Open decisions

- [ ] Confirm Devpost multiple-submission rule for this specific hackathon
- [x] Pick team members / confirm solo build
- [ ] Decide which track to formally submit Household-to-Habitat under (Good Neighbor vs. cross-track framing)
- [ ] Build the eligibility-rules knowledge base dataset (flagged as the piece most likely to eat unplanned time)

## Notes in this vault

- [[Household-to-Habitat]]
- [[Relocation Copilot]]
- [[Build Timeline]]
