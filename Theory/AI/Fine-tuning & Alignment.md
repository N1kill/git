---
tags: [theory, ai, fine-tuning, alignment]
---

# Fine-tuning & Alignment

Related: [[LLM's]] · [[ML]] · [[DL]] · [[Prompt Engineering]]

## Intuition
A pretrained base LLM is really good at "continue this text plausibly" — but that's not the same as "be a helpful, instruction-following, safe assistant." Left alone, a base model given "Write a poem about the ocean" might just as easily continue with more instructions ("Write a poem about the mountains") as it would with an actual poem, because both are plausible continuations of internet-style text. Fine-tuning and alignment are the stages that turn a raw next-token predictor into something that reliably behaves like an assistant.

**Fine-tuning** broadly means continuing to train an already-pretrained model on a smaller, more specific dataset — cheaper than training from scratch because the model already has general language capability, you're just steering/specializing it. **Alignment** specifically means fine-tuning toward human preferences and values — not just "can it do the task" but "does it do the task the way humans actually want," including being honest, avoiding harm, and refusing things it shouldn't do.

The reason alignment needs its own separate techniques (beyond plain supervised fine-tuning) is that "what a human prefers" isn't easily written down as a labeled dataset of correct answers the way "translate this sentence" is — it's comparative and subjective (this response is *better* than that one), which is why preference-based methods (RLHF, DPO) exist instead of just more supervised fine-tuning.

## Reference

**Fine-tuning approaches**
- **Full fine-tuning**: update all model weights on new data — most flexible, most expensive (memory and compute scale with full model size), risk of catastrophic forgetting (losing general capability while overfitting to the new narrow data)
- **PEFT (Parameter-Efficient Fine-Tuning)**: freeze most of the model, train only a small number of additional/modified parameters
  - **LoRA (Low-Rank Adaptation)**: freeze original weight matrices, inject small trainable low-rank decomposition matrices alongside them (`W + BA`, where B and A are much smaller than W) — drastically fewer trainable parameters, easy to swap different LoRA adapters on/off the same base model
  - **QLoRA**: LoRA combined with quantizing the frozen base model to lower precision (e.g. 4-bit) — makes fine-tuning large models feasible on much less GPU memory
  - **Adapters, Prefix-tuning, Prompt-tuning**: other PEFT variants — insert small trainable modules or trainable "soft prompt" vectors rather than modifying core weights directly
- **Instruction tuning**: SFT specifically on (instruction, response) pairs across many varied task types — this is what turns a base model into an "instruction-following" model in the first place, prerequisite to most other fine-tuning

**Alignment pipeline**
1. **SFT (Supervised Fine-Tuning)**: train on curated high-quality (prompt, ideal response) demonstrations, usually written/vetted by humans
2. **Reward Modeling**: collect human preference data — given two model responses to the same prompt, a human picks which is better. Train a separate reward model to predict this preference score
3. **RLHF (Reinforcement Learning from Human Feedback)**: use the reward model as the reward signal in an RL loop (classically PPO — Proximal Policy Optimization) to further tune the policy (the LLM) to produce responses the reward model scores highly, with a KL-divergence penalty against the original SFT model to prevent it from drifting too far / degenerating (reward hacking)
4. **DPO (Direct Preference Optimization)**: reformulates the same preference-alignment goal as a direct supervised loss on preference pairs, mathematically derived to have the same optimal solution as RLHF but without needing a separate reward model or the RL training loop — simpler, more stable, cheaper, and has become the more common approach over classic PPO-based RLHF in many pipelines
5. **Constitutional AI / RLAIF**: use AI-generated feedback (the model critiquing/ranking its own outputs against a set of principles) instead of, or alongside, purely human-generated preference data — reduces the human-labeling bottleneck

**Key failure modes**
- **Catastrophic forgetting**: fine-tuning too aggressively on narrow data degrades general capabilities the base model had
- **Reward hacking**: the policy finds ways to score well on the reward model that don't actually reflect genuine quality/human preference (e.g. learning to be longer or more sycophantic because that scored better, not because it's actually better) — the KL penalty against the SFT model in RLHF exists specifically to limit this
- **Alignment tax**: aligned models sometimes show slightly reduced raw capability/creativity relative to the base model on certain tasks, as a side effect of being tuned toward safety/preference — an active tension labs manage rather than fully solve

**When fine-tuning is worth it vs. not** (see also [[Prompt Engineering]])
- Worth it: need a systematic, hard-to-prompt-elicit behavior change; need to shrink prompts/cost by baking in a pattern; need domain-specific style/format reliably at scale
- Often not worth it: the task is achievable via good prompting or RAG — fine-tuning is more expensive to iterate on, and doesn't fix a lack of factual knowledge as reliably or cheaply as retrieval does

**Evaluation of fine-tuned/aligned models** — see [[Evaluation & Guardrails]] for benchmarks, human eval, and safety testing approaches.
