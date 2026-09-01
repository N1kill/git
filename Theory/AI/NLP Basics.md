---
tags: [theory, ai, nlp, foundations]
---

# NLP — Natural Language Processing Basics

Related: [[LLM's]] · [[Embeddings & Vector DBs]] · [[Transformers]]

## Intuition
NLP is the broader field LLMs now dominate — getting computers to work with human language. The core problem across all of NLP is the same: text is discrete, unstructured, and ambiguous, but ML needs numbers to operate on. Every technique in classical NLP is really a different answer to "how do I turn words into numbers in a way that preserves meaning?" The field's history is a steady move from sparse, hand-built representations (bag-of-words) toward dense, learned representations (embeddings) that capture semantic similarity — and LLMs are the endpoint of that trajectory so far, where the "representation" and the "task model" are the same giant network.

Even though LLMs now handle most of what used to require separate specialized NLP pipelines, the underlying vocabulary (tokens, embeddings, n-grams, parsing) is still the language the field is described in, and still shows up directly in how LLMs work internally.

## Reference

**Text representation, in historical order**
- **Bag-of-Words (BoW)**: represent a document as word counts, ignoring order and grammar. Simple, loses all structure.
- **TF-IDF**: weight word counts by how rare/informative they are across a corpus — `TF-IDF = TF(t,d) × log(N / DF(t))`. Downweights common words like "the", upweights distinctive terms.
- **N-grams**: sequences of n consecutive words/tokens — captures a little local order that BoW loses
- **Word embeddings (Word2Vec, GloVe)**: dense vectors where semantically similar words end up close together in vector space, learned by predicting context words from a target word (or vice versa). Famous property: vector arithmetic captures relationships (`king - man + woman ≈ queen`)
- **Contextual embeddings (ELMo, then BERT/Transformers)**: the same word gets a *different* vector depending on context ("bank" in "river bank" vs "bank account") — a major leap over static word embeddings

**Classic NLP tasks**
- **Tokenization**: splitting text into words/subwords/sentences
- **POS tagging**: labeling each word's part of speech (noun, verb, etc.)
- **Named Entity Recognition (NER)**: identifying and classifying entities (people, orgs, locations, dates) in text
- **Parsing**: building the grammatical structure of a sentence (dependency parsing, constituency parsing)
- **Sentiment analysis**: classifying text as positive/negative/neutral
- **Machine translation**: text in language A → language B
- **Text summarization**: extractive (pick key sentences) vs abstractive (generate new summary text)
- **Named coreference resolution**: figuring out what pronouns/references point to ("she" = "Maria" mentioned two sentences earlier)

**Language modeling, pre-Transformer**
- **N-gram language models**: predict next word from the previous n-1 words using frequency counts — simple, no generalization beyond seen n-grams
- **RNN language models**: predict next word using a hidden state carried through the sequence — better generalization, still sequential/slow, still struggles with long-range dependency (see [[Transformers]] for why this got replaced)

**Evaluation metrics specific to NLP**
- **Perplexity**: how "surprised" a language model is by held-out text — lower is better, mathematically `2^(cross-entropy)`
- **BLEU**: n-gram overlap between generated and reference text — used for translation, has known weaknesses (penalizes valid paraphrasing)
- **ROUGE**: recall-oriented overlap metric, common for summarization
- **METEOR, BERTScore**: attempt to fix BLEU's weaknesses using synonyms/semantic similarity instead of exact n-gram match

**Why this matters even in the LLM era**
Retrieval in RAG systems (see [[RAG]]) still sometimes uses TF-IDF/BM25 (a refined TF-IDF variant) alongside or instead of embeddings, because sparse lexical matching is fast, interpretable, and excellent at exact keyword/name matches that dense embeddings can blur. Real systems often combine both ("hybrid search").
