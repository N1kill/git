---
tags: [theory, ai, rag]
---

# RAG — Retrieval-Augmented Generation

Related: [[Types of RAG]] · [[LLM's]] · [[Embeddings & Vector DBs]] · [[NLP Basics]] · [[Agentic AI]]

## The core idea
An LLM's knowledge is frozen at training time and limited to whatever fit into its weights — it can't know your private documents, and it can't know anything past its training cutoff. You could try to fix this by fine-tuning the model on new information, but that's slow, expensive to update, and models are genuinely unreliable at "storing and retrieving" specific facts that way.

RAG sidesteps the whole problem: instead of trying to bake facts *into* the model's weights, fetch the relevant facts at the moment of the question and hand them to the model *as context*, then let the model do what it's actually good at — reading and synthesizing, not memorizing. This directly attacks hallucination (the model has real facts in front of it instead of guessing) and the staleness problem (update your knowledge source anytime, no retraining needed).

## The pipeline, and where the real difficulty lives
1. **Ingestion**: load documents → break them into **chunks** → embed each chunk → store in a vector DB (or other index — see [[Types of RAG]])
2. **Retrieval**: embed the incoming query with the same embedding model → find the most similar stored chunks
3. **Augmentation**: insert those chunks into the prompt, usually with an instruction like "answer using only the following context"
4. **Generation**: the LLM answers, grounded in what was retrieved

The uncomfortable truth about RAG systems: almost all the real-world quality problems live in step 2, not step 4. If you retrieve the wrong chunks, the LLM will confidently answer from wrong information — it has no way to know the retrieved context is bad, it just trusts what it was handed. This is why chunking strategy and retrieval method matter far more to a RAG system's quality than which LLM sits at the end of the pipeline.

## Chunking — why it's not just "split the text somewhere"
- **Fixed-size chunking**: split every N tokens/characters, usually with some overlap so a concept split across a boundary still appears whole in at least one chunk. Simple, but structure-blind — it'll happily cut a table in half or split a sentence mid-thought.
- **Semantic/recursive chunking**: split at natural boundaries — paragraphs, sections, headers — so each chunk stays coherent on its own.
- **Document-type-aware chunking**: different content types need different splitting logic entirely — code should split on function/class boundaries, tables should stay intact rather than split mid-row, transcripts by speaker turn. Generic fixed-size chunking actively breaks structured content like this.
- **The core tradeoff**: chunks too small lose context and become meaningless in isolation; chunks too large dilute the embedding's specificity and waste context window on irrelevant text. There's no universally "right" size — it depends on how self-contained a unit of meaning naturally is in your source material.

## Improving retrieval quality beyond the basic pipeline
- **Re-ranking**: retrieve a larger, cheap candidate set (say, top 50 via fast embedding similarity), then run a slower but more accurate model over just those candidates to re-score and keep the true top-k. Worth doing in almost any production RAG system — it's a meaningful quality gain for a bounded extra cost, since the expensive step only runs on a small candidate set.
- **Query rewriting**: the user's raw question is often not what you'd want to search with — vague, oddly phrased, missing context. Having the LLM reformulate the query before retrieval often improves what gets found.
- **HyDE (Hypothetical Document Embeddings)**: instead of embedding the question directly, have the LLM first generate a hypothetical *answer*, and embed that instead. A hypothetical answer often resembles the real target document more closely than the question itself does — useful when questions and their answers are phrased very differently.
- **Multi-hop retrieval**: retrieve, read what came back, decide if that's actually enough to answer, and retrieve again if not. Necessary for questions that require synthesizing information scattered across multiple sources rather than sitting in one chunk.

For the different overall *architectures* RAG systems can take — vector-based, graph-based, hybrid, agentic, and more — see **[[Types of RAG]]**, since each represents a genuinely different set of tradeoffs worth understanding on its own.

## Failure modes to watch for
- Retrieving irrelevant chunks → the LLM either ignores them (fine) or gets confused and grounds a wrong answer in them (bad, and hard to detect without checking)
- "Lost in the middle" — stuffing many chunks into a long context can cause the model to underweight information sitting in the middle of that context (see [[LLM's]])
- The answer genuinely isn't in the knowledge base at all — a well-built system should recognize and say so, rather than let the LLM fill the gap with a plausible-sounding hallucination
- Stale index — the source knowledge base was updated but the vector DB wasn't re-indexed, so retrieval confidently returns outdated information

## Evaluating a RAG system
Two genuinely separate things need checking, because a system can be strong at one and weak at the other:
- **Retrieval quality**: did the right chunk actually get retrieved, and ranked highly? (Recall@k, Precision@k, MRR)
- **Generation quality**: is the final answer actually supported by what was retrieved (faithfulness/groundedness), and does it actually address the question (answer relevance)? Frameworks like RAGAS automate this using an LLM as the judge.
