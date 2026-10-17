# 03 · Data and supervision

[Guide home](README.md) · Previous: [Models, tokens, and inference](02-models-tokens-and-inference.md) · Next: [How learning works](04-how-learning-works.md)

## Learning goals

- Distinguish source material, examples, search indexes, and evaluation data.
- Understand what supervision means for a language model.
- Recognize why synthetic examples require acceptance checks and human judgment.

## Four artifacts, four responsibilities

A **raw corpus** contains sources and metadata before task-specific transformation; its documents may already have structured formatting.

**Training examples** pair inputs with desired outputs to change model behavior. A policy alone is not an instruction-response example: a process must select questions, evidence, and desirable responses.

A **retrieval index** makes source passages searchable. Building it does not, by itself, change a language model's parameters.

An **evaluation dataset** contains cases and assessment criteria used to measure behavior rather than teach those same cases. Its purpose differs from training even when its records look similar. [Module 05](05-splits-leakage-and-generalization.md) covers the detailed separation rules.

Think of source records, fixtures, a database index, and acceptance tests: shared domain, different responsibilities. Unlike ordinary fixtures, training examples influence a statistical model without guaranteeing behavior.

## What counts as supervision?

In **supervised learning**, examples provide a desired target. A target might be a category, a number, or an entire assistant response. For an LLM, prose such as “Receipt evidence is missing; request it before approval” is a legitimate target. Supervision is not limited to short labels.

**Unsupervised learning** finds structure without supplied task answers, such as document groupings. **Self-supervised learning** constructs targets from data itself: the following token supplies a next-token prediction target without individual human labeling. These describe learning signals, not absence of human involvement.

Humans choose sources, design tasks, review representative examples, resolve ambiguous labels, and set acceptance criteria. They need not hand-label every item. Automated transformations and teacher models can expand a dataset, but their outputs inherit the assumptions and errors of those processes.

## Worked example: from policy to example

Start with fictional equipment policy version 3: a 30-day return window and required receipt evidence. Preserve its identifier and applicability information in the corpus.

Construct an example with a day-24 request, no receipt, and the relevant policy passage. Its assistant target should distinguish the satisfied deadline from the missing evidence. Add another example with receipt evidence so that the model does not learn “always defer.” Neither example represents genuine legal reasoning.

Citing version 3 locates the source; it does not establish correct interpretation or application. Inventing receipt evidence makes an example defective even with a valid citation. Evaluation must check missing-evidence handling, not merely citation-shaped text.

## Quality before quantity

Cleaning should remove extraction artifacts while preserving meaning, identifiers, and version information. **Provenance** records where content came from and how it was transformed. It supports auditability; it does not replace accuracy checks or permission decisions.

Duplicates overweight repeated statements. Document balance matters: long instruments can crowd out short but important ones. Sampling only easy cases biases learning away from messy requests. Quality requires faithful targets, coverage, and representative difficulty—not just grammar.

**Synthetic data** is generated rather than directly collected as human-authored task responses. A teacher model can generate source-grounded examples; training another model on those outputs is a form of **distillation**. Grounding reduces opportunities for invention but does not eliminate them. Teacher agreement, especially within one model family, can reflect shared errors. Generated answers are not automatically “gold,” meaning trusted reference answers.

## How this applies here

The source collection includes seven selected official UN/ICTY records plus nine historical legal instruments, with 915 article/rule records and 2,511 citations. Machine-verified source provenance is separate from actual operator processing consent. Historical source versions must not be presented as complete current law.

The six planned training slices teach different behaviors:

- **A — Raw-source analysis:** analyze material from the source itself.
- **B — RAFT:** use five TRAIN passages with relevant and distracting context, including zero-evidence cases.
- **C — Distilled case briefs:** learn source-faithful structured summaries.
- **D — Faithful negatives:** recognize insufficient evidence and handle lawful controls appropriately.
- **E — Domain-edge formats:** handle less typical domain formats.
- **F — Citation drills:** practice accurate source attribution.

RAFT trains context use through examples; RAG retrieves external context at inference. LoRA changes model parameters later. These are different operations.

The bounded paid pilot requests 300 training examples—50 per slice—and 50 evaluation examples using same-family Sol/Luna models, with a USD 50 local ledger. Requested counts are not accepted counts. The separate full target is 3,650 accepted training examples and 1,100 evaluation examples and needs separate approval. No generated-output success is established yet.

## Common misconceptions

- “Official sources make every derived answer correct.” Transformations can introduce errors.
- “Synthetic means unusable.” It means validation remains necessary.
- “A search index is a trained LLM.” They are different artifacts.

## Alternative methods

- **Human-authored examples:** direct control, with review still required.
- **Reviewed real interactions:** realistic coverage subject to permissions and privacy.
- **Template-generated examples:** controlled variation, but potentially narrow language coverage.

## Self-check

1. Can an assistant paragraph be a supervised target?
2. Why are 300 requests not 300 accepted examples?
3. What does a valid citation fail to establish?

### Answer key

1. Yes; its desired token sequence provides supervision.
2. Generation can fail, and produced examples can fail acceptance checks.
3. It does not establish that the answer faithfully interprets or applies the cited evidence.
