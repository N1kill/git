---
tags: [theory, ai, evaluation, safety]
---

# Evaluation & Guardrails

Related: [[LLM's]] · [[Fine-tuning & Alignment]] · [[Agentic AI]] · [[RAG]]

## Intuition
Traditional ML evaluation is straightforward because tasks usually have one correct answer: accuracy, F1, MSE — compare prediction to ground truth, compute a number (see [[ML]]). LLM evaluation is harder because most interesting outputs are open-ended text, where "correct" isn't a single string — many different phrasings can be equally good, and quality itself is often subjective (tone, helpfulness, safety) rather than a fact you can check.

This is why LLM evaluation is really several different problems wearing one name: **capability eval** (can it do the task at all — benchmarks), **quality eval** (is this specific response good — human eval or LLM-as-judge), and **safety eval** (does it avoid harmful/unwanted behavior — red-teaming and guardrails). A system can score well on one axis and fail badly on another — a model can ace a reasoning benchmark and still be unsafe, or be perfectly safe and unhelpfully evasive.

**Guardrails** are the operational, runtime counterpart to evaluation — instead of just measuring quality/safety after the fact, guardrails are active mechanisms that catch or prevent bad outputs/inputs during actual use: filtering harmful requests before they reach the model, checking outputs before they reach the user, constraining what actions an agent is allowed to take.

## Reference

**Benchmark-based evaluation**
- Standardized test sets with (mostly) objectively gradable answers, used to compare models
- Examples by domain: MMLU (broad knowledge), HumanEval/MBPP (code generation), GSM8K/MATH (math reasoning), HellaSwag (commonsense), TruthfulQA (resistance to generating popular misconceptions)
- Known weaknesses: benchmark contamination (test data leaking into training data, inflating scores), benchmarks not reflecting real-world task distribution, saturation (top models all near ceiling, losing discriminative power)

**Human evaluation**
- Human raters score or compare model outputs — most reliable for subjective quality (helpfulness, tone, tastefulness) but slow and expensive
- **Pairwise comparison / Elo-style ranking**: raters pick the better of two responses rather than scoring in isolation — generally more reliable than absolute scoring, which raters tend to apply inconsistently
- Used directly in preference-based alignment data collection (see [[Fine-tuning & Alignment]])

**LLM-as-judge**
- Use a strong LLM to evaluate/score another model's outputs, following a rubric — much cheaper and faster than human eval, scales well
- Known biases to watch for: position bias (favoring whichever answer appears first), verbosity bias (favoring longer answers regardless of quality), self-preference bias (a model may rate outputs similar to its own style more favorably)
- Best practice: randomize answer order, provide a clear rubric, and validate periodically against human judgments rather than trusting it blindly

**RAG-specific evaluation** (see [[RAG]] for detail)
- Faithfulness/groundedness: is the answer actually supported by retrieved context, or did the model add unsupported claims?
- Retrieval quality: Recall@k, Precision@k, MRR — separate from generation quality, since a RAG system can retrieve well but generate poorly, or vice versa

**Agent-specific evaluation** (see [[Agentic AI]] for detail)
- Task completion rate in sandboxed/simulated environments
- Efficiency: number of steps/tool calls/tokens used relative to an optimal solution
- Trajectory evaluation: not just did it get the right final answer, but did it take a sensible path there (avoided unnecessary or risky actions along the way)

**Guardrails — input side**
- Content filtering/classification on user input before it reaches the model (detecting harmful requests, PII, prompt injection attempts)
- Rate limiting, input length/format validation

**Guardrails — output side**
- Output classifiers/moderation checking generated content before it's shown to the user or used to trigger an action
- Structured output validation (does the output match the required schema/format before downstream systems consume it)
- Fact-checking/grounding checks for RAG outputs specifically

**Guardrails — action side (specific to agents)**
- Permission scoping: an agent's tools should only be able to do what's actually needed, least-privilege style
- Human-in-the-loop confirmation for irreversible or high-stakes actions (sending messages, spending money, deleting data)
- Hard limits on step count/loop iterations to prevent runaway agent loops (see failure modes in [[Agentic AI]])
- Sandboxing: running risky actions (code execution, file system access) in an isolated environment rather than directly against production systems

**Red-teaming**
Deliberately probing a model/system with adversarial inputs to find failure modes before real users do — jailbreak attempts, edge cases, harmful-request variations. Distinct from routine benchmark eval because it's specifically adversarial and open-ended rather than measuring against a fixed test set.

**Why no single metric suffices**
Optimizing hard for one metric (e.g. a benchmark score, or "helpfulness" alone) tends to create blind spots elsewhere (Goodhart's law in practice) — real evaluation pipelines combine multiple methods above, layered, rather than trusting any one signal in isolation.
