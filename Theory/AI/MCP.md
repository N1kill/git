---
tags: [theory, ai, mcp, agents]
---

# MCP — Model Context Protocol

Related: [[Agentic AI]] · [[LLM's]] · [[RAG]]

## Intuition
Before MCP, every application that wanted to connect an LLM to an external system (Slack, GitHub, a database, your filesystem) had to write custom, one-off integration code — the tool definitions, the auth handling, the request/response glue — for that specific model provider's function-calling format. This is the classic "M×N problem": M applications × N tools/data sources = M×N custom integrations, none of them reusable.

MCP (introduced by Anthropic) standardizes this into a common protocol: any MCP-compatible **client** (an LLM app) can talk to any MCP-compatible **server** (a tool/data provider) without custom glue code, the same way HTTP lets any browser talk to any web server. Build one MCP server for GitHub, and every MCP-compatible AI app can use it — not just yours.

The mental model: MCP servers expose three kinds of things to a model — **tools** (functions the model can call to take action), **resources** (data the model can read, like files or query results), and **prompts** (reusable prompt templates the server provides). The client (the AI application) discovers what's available from a connected server and offers it to the model, which decides when and how to use it — same underlying tool-use mechanism as any agent (see [[Agentic AI]]), just standardized at the transport/discovery layer.

## Reference

**Architecture**
- **Host**: the AI application itself (e.g. Claude Desktop, an IDE, a custom app)
- **Client**: lives inside the host, maintains a 1:1 connection to a single MCP server
- **Server**: a lightweight program that exposes tools/resources/prompts for a specific system (e.g. "GitHub MCP server," "Postgres MCP server")
- A host can connect to many servers simultaneously, each via its own client instance

**Core primitives exposed by a server**
- **Tools**: functions the model can invoke (e.g. `create_issue`, `search_files`) — model-controlled, the model decides when to call these
- **Resources**: readable data (files, DB rows, API responses) — can be application-controlled (attached automatically) or model-requested
- **Prompts**: pre-written prompt templates the server offers, often user-triggered (e.g. a slash command) rather than model-decided

**Transport**
- **stdio**: server runs as a local subprocess, communicates over stdin/stdout — used for local tools (filesystem, local scripts)
- **HTTP/SSE (Server-Sent Events)** or newer **Streamable HTTP**: used for remote servers — the server can live anywhere, client connects over the network
- Underlying message format is JSON-RPC 2.0 in both cases

**Discovery and invocation flow**
1. Client connects to server, server advertises its available tools/resources/prompts (with schemas)
2. Host surfaces these to the model as available tools (same shape as native function-calling)
3. Model decides to call a tool → client sends the call to the server → server executes → result returned to client → fed back into model's context
4. This loop is identical in shape to the agentic tool-use loop in [[Agentic AI]] — MCP just standardizes steps 1 and 3 so they aren't reinvented per integration

**Why this matters over "just use function calling directly"**
Function calling (the model's ability to output a structured tool call) is a model-level capability that already existed. MCP doesn't replace that — it standardizes everything *around* it: how tools are described, discovered, connected to, and authenticated against, so that tool integrations become portable, shareable packages instead of app-specific code every developer rewrites.

**Security considerations**
- A malicious or compromised MCP server can expose harmful tools, or return content designed to manipulate the model (prompt injection via tool results/resources)
- Because resource content flows directly into the model's context, untrusted resource content should be treated with the same suspicion as untrusted user input, not blindly trusted as "system-provided"
- Auth/permission scoping matters — a server should only be able to do what its declared tools legitimately need

**Relationship to Agentic AI and RAG**
MCP is an integration/transport standard, not a reasoning framework — the actual decision-making loop (when to call a tool, how to use results) is still the agentic loop described in [[Agentic AI]]. Retrieval-based tools exposed over MCP (e.g. a vector-search server) are one way an [[RAG]] pipeline can be wired into an agent without custom integration code.
