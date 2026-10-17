# Module 06 — Embeddings, indexing, RAG and RAFT

[Guide home](README.md) · [Previous: splits](05-splits-leakage-and-generalization.md) · [Next: fine-tuning](07-fine-tuning-lora-and-export.md)

## What you should understand

- How text becomes searchable by meaning as well as words.
- Why building an index is neither model training nor dataset assembly.
- The difference between RAG at answer time and RAFT-style examples during training.
- Why retrieval success and answer correctness must be measured separately.

## 1. An embedding is a representation, not a fact database

An **embedding** is a vector: an ordered list of numbers representing an input in a learned space. An embedding model is trained so that certain relationships between inputs become useful relationships between vectors.

For retrieval, two differently worded passages about a similar issue may receive similar vectors. “Can I send the device back?” may match “Returns accepted within 30 days” despite limited exact word overlap.

This is a learned approximation. Similarity does not mean truth, legal applicability or logical entailment. A passage denying eligibility and a passage granting eligibility may both be highly similar to an eligibility question because they discuss the same topic.

The retrieval embedding model and the answer-generating model have different jobs. In this project, **BGE** represents passages for search; **Qwen** is the intended student answer generator. Running BGE over documents does not fine-tune Qwen.

## 2. From documents to an index

Indexing is closer to creating a database search index than to training a new application model:

1. Divide source text into manageable passages while retaining source identity and location.
2. Run a fixed embedding model on each passage to obtain vectors.
3. Store those vectors with a mapping back to passage text and provenance.
4. Build structures that support efficient similarity searches.
5. At query time, encode the question with the compatible embedding model and retrieve candidate passages.

The embedding model's parameters stay fixed during this process. **Computing embeddings** is inference with that model. **Training an embedding model** would be a separate learning task with its own data and objective.

Passage size is a trade-off. Very small passages can lose necessary conditions; very large passages can mix topics and consume the answer model's context window. Overlap reduces boundary loss but creates redundancy. This project uses approximately 350-word retrieval passages with 50-word overlap; these are not the same as its larger source-splitting chunks.

## 3. Why hybrid retrieval?

**Sparse retrieval**, here BM25, scores lexical matches using term statistics. It is useful for exact article numbers, names and uncommon terms. It does not necessarily recognize every paraphrase.

**Dense retrieval** uses embeddings and vector similarity. It can find semantic matches, but may confuse closely related topics or miss a crucial identifier.

**Hybrid retrieval** combines their signals. This project's configured blend gives BM25 and dense scores weights of 0.4 and 0.6. Those are retrieval settings, not 40% and 60% confidence that the answer is true. Their suitability is an evaluation question, not a universal best ratio.

The vector index uses FAISS; normalized embeddings support similarity-based ranking. The normal workflow selects up to five passages. The index must remain bound to the exact passage order, contents and embedding model version: changing the mapping can attach a correct-looking vector to the wrong evidence.

The recorded selected library has **8,347 passages**. Building all of their dense vectors is the CPU job discussed in the project status. Its completion does not create the final supervised training dataset. The current encoder saves embeddings only after the full encode, which is why an early restart lost partial computation.

## 4. RAG changes the input, not the weights

**Retrieval-augmented generation (RAG)** supplies retrieved material as part of an answer model's prompt.

```mermaid
flowchart LR
    Q["User question"] --> R["Retrieve from versioned library"]
    R --> P["Question + selected evidence + instructions"]
    P --> M["Answer model with fixed weights"]
    M --> A["Answer and citations"]
```

For the support example, the model receives a question, the applicable policy passage and perhaps the receipt details. It generates an answer conditioned on that context. After the request, the new policy has not automatically become a permanent weight update.

This separation is valuable when knowledge changes frequently. Update the permitted source library and rebuild affected retrieval artifacts instead of assuming every policy revision needs another fine-tuning run. Still test retrieval, version selection and answer behavior after an update.

## 5. RAFT teaches behavior with retrieved-looking context

**Retrieval-augmented fine-tuning (RAFT)** in this project means training examples that ask the student to answer while reading a mixture of useful and distracting evidence. It is supervised practice in context use, not a separate magic model type.

For a support question about returns:

- One passage may contain the actual return conditions.
- Other passages may discuss repairs, shipping or a different product.
- The target answer should use the supporting passage, not blend unrelated conditions into a new policy.

This project's Slice B uses **five unique eligible TRAIN passages**. Positive examples include an evidenced gold passage and four other-document distractors. Some examples contain no supporting passage, so the appropriate target is insufficient evidence rather than a fabricated answer. Checks bind the rendered passages, IDs, provenance and exact quote attribution.

**Gold** means designated supporting evidence for the example; designation still requires verification. A distractor from another document is not guaranteed irrelevant—it could independently support the answer. Structural validation cannot settle all semantic relevance questions.

RAG and RAFT are complementary but separable:

- A base model can use RAG without any RAFT training.
- A fine-tuned model can answer without retrieval, although it cannot consult missing context.
- RAFT training does not guarantee that the production retriever finds the right evidence.

## 6. Two failure layers

**Retrieval failure:** the needed policy was not among the returned passages. Investigate indexing, filters, chunking, versions and ranking.

**Generation failure:** the needed policy was present, but the model misread it, ignored a condition or cited an unrelated passage. Investigate instructions, examples, model behavior and task difficulty.

A real citation only establishes that a reference exists. It does not establish that the cited passage supports the conclusion.

## Alternatives worth recognizing

- **BM25 alone:** inexpensive and interpretable; often a strong exact-term baseline.
- **Dense retrieval alone:** useful for paraphrases; weaker on some identifiers and exact matches.
- **Reranking:** a second model scores retrieved candidates more carefully; adds latency and compute.
- **Long-context prompting:** includes more source material directly; simpler retrieval logic, but higher token cost and no guarantee that important evidence is used.

## Self-check

1. Does adding a document to the index change Qwen's parameters?
2. A response is wrong even though the correct passage was retrieved. Is rebuilding the index necessarily the fix?
3. What behavior should a no-support RAFT example teach?

**Answers:**

1. No. Indexing changes the external searchable library, not Qwen's weights.
2. No. Diagnose the answer-generation layer and the sufficiency/clarity of the provided evidence first.
3. Recognize that the available evidence is insufficient rather than inventing missing support. This is not the same as concluding that the proposition is false.

**Professional phrasing:** “We combine hybrid retrieval at inference with supervised context-grounding examples during adaptation, and evaluate retrieval and answer quality separately.”
