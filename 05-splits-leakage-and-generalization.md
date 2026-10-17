# Module 05 — Splits, leakage and generalization

[Guide home](README.md) · [Previous: learning](04-how-learning-works.md) · [Next: retrieval](06-embeddings-rag-and-raft.md)

## What you should understand

- Why training success and useful performance are different claims.
- What training, validation and test sets are allowed to influence.
- How a model can appear excellent because an experiment accidentally gave it the answers.
- What this project's held-out regions do—and do not—establish.

## 1. The real goal is generalization

**Generalization** is performing usefully on relevant inputs that were not used to fit or select the system. Memorizing the training set is usually not the product requirement.

Imagine training a support assistant on return requests. If it learns “approve requests written in this template,” it may score well on similar examples while failing a differently worded request with the same facts. You wanted it to apply the policy; it learned a shortcut correlated with the labels.

In developer terms, passing tests copied from the implementation's own examples is weaker evidence than passing independent acceptance tests. The analogy has limits: ML performance is usually statistical, and representative coverage matters as much as individual assertions.

A **distribution** describes which kinds of inputs and outcomes occur, and how often. The next customer's requests may differ from the historical requests. This change is called **distribution shift**. A test set is useful only to the extent that it represents the claim you want to make.

## 2. Three datasets, three jobs

| Set | What it is for | What may depend on it |
|---|---|---|
| Training | Fit model parameters using example targets | Gradient updates and the learned adapter |
| Validation/development | Choose settings, checkpoints and design changes | Learning rate, number of epochs, prompts and other development decisions |
| Final test | Estimate performance after development choices are fixed | The reported acceptance result—not another round of tuning on the same test |

Evaluating a validation example does not normally update weights directly. But choosing a checkpoint because it scored best on validation still lets validation influence the resulting system. That is its intended purpose.

If you repeatedly inspect final-test failures and adapt the system to those exact cases, that set has effectively become development data. Freeze a new test set for a fresh acceptance claim. Calling an overused dataset “test” does not restore independence.

## 3. Leakage is an information-boundary problem

**Leakage** means information reaches training or model selection through a route that invalidates the intended evaluation.

A support-policy example:

1. One original request is paraphrased into ten training examples.
2. A random row split puts eight paraphrases in training and two in test.
3. The test appears to measure unseen requests, but mostly measures recognition of the same underlying case.

The right split unit may be the original request, document, customer, organization or time period—not the row. **Group splitting** keeps related records together. **Time splitting** trains on earlier records and evaluates later ones when future performance is the real target.

Other leakage routes include:

- A generated assistant answer quoting a held-out paragraph even though the user prompt does not.
- A summary of the full source document being used to generate training data from a supposedly restricted region.
- Hidden evaluation answer keys accidentally included in inference prompts.
- Fitting a data-dependent preprocessing step on all examples before the split.

Searching for identical text helps, but paraphrases and shared events can leak without exact matches.

## 4. Two different split boundaries in this project

**Source boundary:** the selected corpus is divided into TRAIN and TEST regions, with buffer chunks around boundaries. The recorded selected split has **1,758 TRAIN chunks, 314 TEST chunks and 14 buffer chunks**. Training generation should use eligible TRAIN material; source-grounded evaluation questions use held-out TEST material.

**Student-training boundary:** the intended LoRA training also reserves approximately **5% of the accepted generated examples for validation**. That helps observe training behavior and choose a checkpoint. It is not the same object as the source-region TEST set, and it is not automatically a document-independent final test.

Buffer chunks reduce direct overlap caused by sliding windows. They cannot remove all semantic overlap between nearby passages or related documents. Several records may describe the same underlying facts. A randomly held-out region within a known document supports a narrower claim than an entirely unseen document or institution would.

Another boundary predates this project: the Qwen starting checkpoint might already have encountered public source material during its original training. We cannot conclude “the base model has never seen this” merely because our own TRAIN split excludes it.

## 5. Retrieval access is not automatically leakage

At inference, an open-book RAG system is supposed to retrieve evidence. It may legitimately retrieve a held-out source passage that answers a held-out question. That tests whether the system can **find and use available evidence**, not whether it memorized the answer without access to that evidence.

The prohibited shortcut is supplying the evaluation's hidden reference answer or grading key. Keep the question, allowed evidence and hidden key conceptually separate.

Consequently:

- Plain evaluation asks what the model can do without retrieved passages.
- RAG evaluation asks what the system can do with the permitted library.
- A stronger unseen-document study would require an additional explicitly designed split.

## Alternatives worth recognizing

- **Grouped train/test splits:** better separation of related cases; may leave fewer independent groups.
- **Temporal evaluation:** useful when policies or customer behavior change over time.
- **Cross-validation:** repeats training across partitions to assess variability; expensive for large models and not a cure for bad grouping.
- **External evaluation set:** different source or organization; stronger transfer evidence, but may test a different task distribution.

## Self-check

1. Training loss falls while validation loss rises. What is a likely concern?
2. Is a TEST passage retrieved by a RAG system necessarily leakage?
3. Why might 1,000 questions from seven documents contain less independent evidence than 1,000 unrelated cases?

**Answers:**

1. Overfitting is a concern: the system may be fitting training-specific patterns rather than improving generalization. Investigate data, checkpoints and metrics; one curve is not a complete diagnosis.
2. No. It can be the permitted evidence in an open-book task. Hidden answer-key exposure or violating the declared training/evaluation boundary would be the problem.
3. Questions can share facts, style and source-specific artifacts. Their errors are correlated, so the raw question count overstates the number of independent situations tested.

**Professional phrasing:** “Our source-region holdout limits direct training overlap; it does not establish document-level independence or absence of base-model pretraining exposure.”
