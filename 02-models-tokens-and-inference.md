# 02 · Models, tokens, and inference

[Guide home](README.md) · Previous: [Problem framing](01-problem-framing.md) · Next: [Data and supervision](03-data-and-supervision.md)

## Learning goals

- Place language models within the wider AI landscape.
- Explain weights, tokens, attention, and generation without treating them as fact checking.
- Distinguish training from inference and context from stored parameters.

## Where an LLM fits

**Artificial intelligence (AI)** is the broad category of systems performing tasks associated with intelligent behavior. Not all AI uses machine learning: an explicitly programmed planning or rules system can also qualify. **Machine learning (ML)** learns behavior from data. **Deep learning** is ML using neural networks with multiple processing layers.

A **transformer** is a neural-network architecture using attention. A **large language model (LLM)** is a large model trained on language. Modern LLMs commonly use transformers, but transformers also process non-language data. These terms describe relationships, not rankings.

## Weights are learned settings, not a fact table

A model's **parameters**, often called weights, are numerical settings adjusted during training. Their collective values influence how an input becomes an output. “7B” denotes roughly seven billion parameters; it does not mean seven billion facts, documents, or reliable rules.

Think of a configurable text-processing function: training adjusts settings; inference calls it. Unlike ordinary configuration, learned settings interact across layers. They are not named options such as “knows return policy,” nor individually identifiable facts.

## Tokens, embeddings, and attention

A **token** is a tokenizer-produced unit. Tokens may represent words, word fragments, punctuation, or other text units. A word can require several tokens; token counts also vary across languages and formatting. Therefore, a 1,000-token limit is not a 1,000-word limit.

The model converts tokens into **embeddings**: lists of numbers that let its layers operate on text. These representations carry learned relationships rather than human-readable definitions. Their values become sensitive to context as processing proceeds. External search embeddings are related but serve a different role, covered in [Module 06](06-embeddings-rag-and-raft.md).

**Attention** lets the model combine information from available token positions. In “the receipt is missing, but the purchase date is known,” relevant words can influence how later representations are formed. In a causal language model, a position cannot attend to future tokens that have not been generated. Attention is not conscious focus, and attention values alone are not a dependable explanation of a decision.

## Generation is not verification

An autoregressive model estimates next-token possibilities, selects a token, appends it, and repeats. Answers emerge from successive choices, not necessarily from stored paragraphs.

The process is optimized to produce plausible continuations. It does not inherently check a policy database, verify a citation, or establish that a conclusion follows from authoritative evidence. A written explanation is another generated output, not a direct view into the model's internal reasoning.

## Worked example: a supplied policy

Supply a fictional equipment policy, version 3: returns within 30 days require receipt evidence. Add a request made on day 24 without a receipt. The model might generate, “The deadline is satisfied, but receipt evidence is still needed.” That answer is useful because it matches the supplied conditions, not because token generation proved the conditions true.

An obsolete version waiving receipts may change the answer. Fluency cannot establish which version governs. The application must supply suitable sources and validate results. This support policy is not genuine legal analysis.

## Training stages versus inference

**Pretraining** learns broad patterns from large datasets, often by predicting withheld or next tokens. **Instruction tuning** further trains a model on examples of responding to instructions. Neither stage guarantees truthfulness. **Inference** runs an existing checkpoint to produce outputs without normally updating its learned parameters.

A **checkpoint** is a saved model state. The **context window** is the bounded amount of token context the model can process for a request; input and generated output must fit the applicable limits. Parameters persist between ordinary calls, while supplied context is not automatically learned into those parameters.

**Temperature** adjusts the next-token sampling distribution. Lower values generally concentrate choices; higher values broaden them. Temperature is not a truthfulness control. Reproducibility also depends on sampling settings, random seeds, model revision, software, and execution environment. Even a low temperature is not a universal guarantee of identical results.

## How this applies here

Qwen/Qwen2.5-7B-Instruct is already instruction-tuned. The planned v2 adapter starts fresh from that checkpoint, not from v1. Intended v2 training uses BF16 LoRA; historical v1 used QLoRA. [Module 07](07-fine-tuning-lora-and-export.md) explains these training choices.

Building the external BM25+BGE retrieval index on the Azure CPU VM does not train Qwen's weights. Dense-index readiness was still pending in the supplied snapshot, not a live status report. No generated-output or fine-tuning success has been established yet.

## Common misconceptions

- “More parameters mean more verified facts.” Parameter count is not a reliability measure.
- “The prompt permanently teaches the model.” Ordinary inference leaves weights frozen.
- “A reasoning trace reveals internals.” Generated explanations are not internal instrumentation.

## Alternative methods

- **Templates:** predictable wording when inputs and decisions are structured.
- **Smaller language models:** potentially simpler deployment, requiring task-specific evaluation.
- **Extractive systems:** return source text rather than generating a new explanation.

## Self-check

1. Is every AI system an ML system?
2. How does context differ from parameters?
3. Does lowering temperature verify an answer?

### Answer key

1. No; explicitly programmed AI methods also exist.
2. Context supplies request-time tokens; parameters are learned numerical settings.
3. No; it changes sampling, not source verification.
