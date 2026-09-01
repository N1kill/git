---
tags: [theory, ai, ml, reinforcement-learning]
---

# Reinforcement Learning

Related: [[ML]] · [[LLM's]] · [[Fine-tuning & Alignment]]

## The core idea
RL is fundamentally different from supervised/unsupervised learning because there's no fixed dataset to learn from at all. Instead, an **agent** takes **actions** in an **environment**, receives a **reward** signal telling it how good or bad the outcome was, and gradually learns a **policy** — a strategy for which action to take in which situation — that maximizes reward over time.

The core difficulty that doesn't exist in supervised learning: a reward might arrive many steps after the action that actually caused it (you make a good opening move in a game, but only find out you won 40 moves later). This is the **credit assignment problem**, and most of RL's machinery exists to solve it — figuring out *which* past action deserves credit for a reward that showed up later.

## The vocabulary, built up conceptually
- **State (s)**: the current situation the agent observes
- **Action (a)**: what the agent can do from that state
- **Reward (r)**: the feedback signal for having taken that action
- **Policy π(a|s)**: the agent's strategy — given a state, what action does it take (can be deterministic or a probability distribution over actions)
- **Value function V(s)**: not just the immediate reward, but the *expected total future reward* from being in state s and following the current policy from there — this is what actually lets the agent think ahead instead of being short-sighted
- **Q-function Q(s,a)**: like the value function, but for a specific action from that state — "how good is it to take action a right now, in state s"
- **Discount factor γ**: how much the agent values future rewards vs. immediate ones — closer to 1 means "think long-term," closer to 0 means "grab what's in front of you"

**The Bellman equation** ties all of this together: `Q(s,a) = r + γ·max Q(s',a')` — the value of an action now is its immediate reward, plus the discounted value of the best action you can take next. This recursive structure is what most RL algorithms are actually solving.

## On-policy vs off-policy — why this distinction matters
- **On-policy** (e.g. SARSA): the agent learns only from actions it actually took while following its current policy.
- **Off-policy** (e.g. Q-learning): the agent can learn from any data, including actions taken by a different (even past, or exploratory) policy.
- **Why it matters practically**: off-policy methods can reuse old experience (more sample-efficient, can learn from data the current policy wouldn't have generated), which matters a lot when interacting with the real environment is expensive or slow.

## Where RL actually shows up in AI you'll use day to day: RLHF
The most relevant application of RL for anyone working with LLMs isn't game-playing — it's **RLHF (Reinforcement Learning from Human Feedback)**, the technique used to align LLMs after pretraining.
- A **reward model** is trained on human preference data (given two model responses, which did a human prefer) — this reward model stands in for "reward" since there's no ground-truth score for "good response."
- The LLM (acting as the **policy**) is then fine-tuned using RL (classically PPO) to produce responses that score highly according to that reward model.
- A penalty against drifting too far from the original model is included, to stop the policy from "gaming" the reward model in ways that don't reflect genuine quality (reward hacking).
- **DPO** has largely replaced classic RLHF-via-PPO in many modern pipelines — it achieves a mathematically equivalent result without needing a separate reward model or the RL training loop, making it simpler and more stable. See [[Fine-tuning & Alignment]] for the full alignment pipeline this fits into.

## Use when
RL is the right framing when there's no fixed "correct answer" dataset — only a sequence of decisions and delayed consequences you can score. It's overkill (and much harder to get working) for problems that supervised learning can solve directly; reach for it specifically when the problem is inherently sequential/interactive (games, robotics, resource allocation over time, aligning a model's behavior against a preference signal rather than a fixed label).
