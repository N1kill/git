---
tags: [theory, ai, agents]
---

# Agentic AI

Related: [[LLM's]] · [[RAG]] · [[MCP]] · [[Prompt Engineering]]

## Intuition
A plain LLM call is a single turn: prompt in, text out. An **agent** wraps an LLM in a loop that lets it take actions in the world, observe the results, and decide what to do next — repeatedly — until it decides the task is done. The LLM stops being just a text generator and becomes the "reasoning engine" driving a loop of: think → act → observe → think again.

This matters because a huge class of real tasks can't be solved in one shot: "book me a flight" requires searching, comparing, maybe asking a clarifying question, then executing a booking — each step's result changes what the next step should be. An agent architecture gives the LLM **tools** (functions it can call — search the web, run code, query a database, call an API) and lets it choose which tool to use, with what arguments, based on the current state of the task.

The core tension in agent design is **autonomy vs. reliability**. The more decisions you let the model make on its own (which tool, how many steps, when to stop), the more capable the system is on novel tasks — but also the more ways it can go wrong (infinite loops, wrong tool choice, compounding errors across steps, doing something destructive without checking). Most of agent engineering is about constraining that autonomy just enough — good tool design, clear stopping conditions, human-in-the-loop checkpoints — without strangling the flexibility that makes agents useful in the first place.

## Reference

**Core loop (ReAct pattern — Reason + Act)**
1. **Thought**: model reasons about what to do next, in natural language
2. **Action**: model chooses a tool and generates arguments for it
3. **Observation**: tool executes, result is fed back into the model's context
4. Repeat until the model decides it has enough information to produce a final answer, or a stop condition is hit

**Tool use / function calling**
- The model is given a set of tool definitions (name, description, parameter schema)
- Based on the prompt/context, the model outputs a structured call (tool name + arguments) instead of, or alongside, plain text
- The calling system executes the actual tool (this is NOT done by the LLM itself — the LLM only decides *what* to call) and returns the result to the model
- Tool descriptions matter enormously — the model chooses tools based on how well the description matches the task, ambiguous/overlapping tool descriptions cause wrong tool selection

**Planning strategies**
- **Zero-shot/reactive**: decide the next single action based only on current state, no explicit upfront plan (this is plain ReAct)
- **Plan-and-execute**: generate a full multi-step plan upfront, then execute steps (with optional re-planning if a step fails or reveals new information) — better for tasks with predictable structure, less adaptive to surprises
- **Tree-of-thought / multi-path exploration**: explore multiple reasoning branches, evaluate, backtrack — expensive but better for tasks with many possible approaches where the first idea often isn't the best one

**Memory in agents**
- **Short-term/working memory**: the current context window — conversation so far, recent tool results
- **Long-term memory**: persisted across sessions, usually via a vector DB (see [[Embeddings & Vector DBs]]) or structured store — retrieved into context when relevant, since it can't all fit in-context permanently
- Without memory management, long-running agents either lose important early context (falls out of the window) or accumulate so much context that cost/latency/attention quality degrade

**Multi-agent systems**
- Multiple LLM agents, each often with a specialized role (e.g. planner, researcher, coder, reviewer), collaborating on a task
- **Orchestrator/worker pattern**: one agent breaks down the task and delegates to specialist sub-agents, then synthesizes their outputs
- **Debate/critique patterns**: agents review or challenge each other's outputs to catch errors — a form of self-correction at the system level
- Tradeoff: more agents = more perspectives and specialization, but more coordination overhead, cost, and latency, plus new failure modes (agents talking past each other, compounding hallucinations)

**Failure modes specific to agents**
- **Compounding errors**: a wrong result at step 2 poisons every step after it, since each step's context includes prior (possibly wrong) results
- **Infinite/repetitive loops**: model keeps calling the same tool or retrying a failed action without changing approach — needs explicit loop/step limits
- **Tool misuse**: wrong tool chosen, or right tool with malformed/hallucinated arguments
- **Over-agency**: taking an irreversible action (sending an email, deleting data, making a purchase) without adequate confirmation — this is why human-in-the-loop checkpoints matter for consequential actions
- **Context window exhaustion**: long agent runs accumulate huge histories of thoughts/actions/observations, eventually exceeding context or degrading quality (see "lost in the middle" in [[LLM's]])

**Evaluating agents**
Harder than evaluating single LLM calls — need to measure task completion rate, efficiency (steps/cost taken vs. optimal), and safety (did it avoid destructive/irreversible mistakes), not just output quality of one response. Often evaluated in sandboxed/simulated environments before being trusted on real systems.

**Relationship to RAG and MCP**
Retrieval (see [[RAG]]) is really just one specific tool an agent can call. [[MCP]] standardizes *how* tools/resources are exposed to a model, so the same tool integrations can be reused across different agent frameworks instead of writing custom integration code for each one.
