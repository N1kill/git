---
tags: [theory, ai, prompt-engineering]
---

# Prompt Engineering

Related: [[LLM's]] · [[Agentic AI]] · [[RAG]]

## Intuition
An LLM's entire "understanding" of a task, for any given call, comes from what's in its context window — there's no separate configuration step, no persistent state, nothing. The prompt IS the interface. Prompt engineering is the practice of structuring that input so the model reliably does what you actually want, exploiting what's known about how these models respond to instruction, structure, and examples.

The reason this is even a skill (not just "ask nicely") is that LLMs are pattern-completion engines at heart: they continue text in ways statistically consistent with their training distribution. A vague or ambiguous prompt gives the model many plausible completions to choose between, and it may pick one you didn't want — not because it "misunderstood" in a human sense, but because the prompt genuinely underspecified the task. Good prompting narrows that space of plausible completions down to the one you actually need.

Two ideas underlie almost every prompting technique: (1) **give the model room to think before answering** (chain-of-thought) — for anything nontrivial, forcing an immediate answer skips the "work" a step-by-step derivation would otherwise do, and (2) **show, don't just tell** (few-shot examples) — demonstrating the exact input/output pattern you want is often more reliable than describing it abstractly.

## Reference

**Core techniques**
- **Zero-shot**: just ask, no examples — works fine for tasks close to the model's training distribution
- **Few-shot**: provide 2-5 example (input, output) pairs in the prompt before the real query — shows the model the exact pattern/format expected
- **Chain-of-Thought (CoT)**: instruct the model to reason step by step before giving a final answer ("think step by step") — significantly improves performance on multi-step reasoning, arithmetic, logic
- **Zero-shot CoT**: just appending "let's think step by step" without any examples, shown to unlock much of CoT's benefit on its own
- **Self-consistency**: sample multiple CoT reasoning paths for the same question, take the majority-vote answer — trades compute for accuracy on reasoning tasks
- **Least-to-most prompting**: break a complex problem into ordered sub-problems, solve sequentially, each building on the previous answer

**Structuring a prompt**
- Clear, explicit task instructions, stated directly rather than implied
- Role/persona framing ("You are an expert X") — can shift tone/register but shouldn't be relied on for factual capability changes
- Explicit output format specification (JSON schema, XML tags, markdown structure) — models follow explicit format instructions much more reliably than implicit ones
- Delimiters (XML tags, triple quotes, markdown headers) to clearly separate instructions from data/context — reduces the model conflating "text to process" with "instructions to follow"
- Positive AND negative examples ("do this / don't do this") — negative examples alone are less reliable than showing the corrected positive version

**Context ordering**
- Put static/stable content (system instructions, reference docs) before dynamic content (the actual query) when caching matters (see [[Cache]])
- For long contexts, critical instructions are often best placed at both the start and end, since middle-of-context information tends to get relatively underweighted (see "lost in the middle" in [[LLM's]])

**Common failure patterns and fixes**
- **Ambiguous instructions** → model picks a plausible-but-wrong interpretation. Fix: be explicit, give constraints, provide an example of correct output
- **Prompt injection**: if untrusted external content (web pages, documents, tool outputs) is included in context, it can contain text that looks like instructions and gets treated as such by the model. Mitigation: clearly delimit untrusted content, instruct the model explicitly to treat it as data not instructions, never let untrusted content alone drive irreversible actions
- **Over-constraining**: too many rules/exceptions in one prompt can cause the model to drop or conflate constraints — simpler, more modular prompts (or breaking into multiple steps/calls) often outperform one giant mega-prompt

**Prompt engineering vs fine-tuning vs RAG — when to use which**
- Prompting: fastest to iterate, no training needed, good for format/behavior/style control and for tasks the model can already mostly do
- RAG (see [[RAG]]): needed when the model lacks access to specific facts/knowledge it should ground answers in
- Fine-tuning (see [[Fine-tuning & Alignment]]): needed when you require a *systematic* behavior/style/format shift that's hard to reliably elicit via prompting alone, or when you need to bake in behavior cheaply at inference time (shorter prompts, lower latency, no need to re-teach the pattern every call)

**Evaluating prompts**
Prompt changes should be evaluated against a fixed test set of representative inputs, not eyeballed on one or two examples — small wording changes can have outsized, sometimes counterintuitive effects on output quality, especially near decision boundaries in ambiguous tasks.
