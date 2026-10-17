# Module 09 — The ML lifecycle: artifacts, compute and reproducibility

[Guide home](README.md) · [Previous: evaluation](08-evaluation-and-human-judgment.md) · [Next: your next project](10-designing-your-next-project.md)

## What you should understand

- What each stage produces and what its success does not prove.
- Why ML systems version data and evaluations as well as code.
- How to think about compute, paid generation, pilots and release gates.
- Why delivery continues after a model file is produced.

## 1. Think in artifacts and contracts

A software build transforms source code into an executable. An ML workflow transforms several kinds of evidence into several different artifacts. Calling everything “the dataset” obscures important dependencies.

| Stage | Main output | What success alone does not prove |
|---|---|---|
| Define task and permitted scope | Task specification and source/use decisions | That the chosen task is valuable |
| Prepare sources and split regions | Traceable source chunks and split assignments | That every source statement is correct |
| Build the evidence library | Searchable passages and hybrid index | That relevant evidence will always be retrieved |
| Generate and validate examples | Candidate and accepted supervised examples | That accepted targets are all semantically correct |
| Adapt the student | Adapter weights and training records | That the student improves over its baseline |
| Merge and export | Loadable deployment artifacts | That runtime outputs preserve measured quality |
| Evaluate and review | Comparable responses, scores and review evidence | That the results generalize to every future use |
| Deploy and monitor | Versioned service and operational observations | That the system can remain unchanged forever |

Evaluation has its own preparation branch. Questions, references and grading rubrics should not simply be reconstructed from the student's answers after seeing how well it did.

## 2. Three distinct compute workloads

**Indexing** uses an embedding model over source passages. In the present preparation run, the dense encoding work is CPU-bound on the remote VM. GPU acceleration could speed a suitably configured encoder, but indexing is not intrinsically a GPU-training operation.

**Synthetic generation** sends prompts to teacher APIs and records responses. It consumes paid input/output tokens and depends on external service availability. It creates candidate training or evaluation records, not Qwen weight updates.

**Student training** performs forward and backward computation to update the adapter. The intended v2 path needs an appropriate GPU environment. The local laptop hosting this repository is not automatically performing all of these jobs just because it initiated them.

Inference after deployment is another workload: serving a trained model and, for RAG, retrieving passages for each user request.

## 3. Reproducibility requires more than a random seed

A **reproducible experiment** preserves enough information to understand and, within stated limits, repeat what happened. Record:

- Source versions, extraction methods and permitted uses.
- Exact train/validation/test assignments and example lineage.
- Prompt templates, teacher deployment identities and generation settings.
- Candidate outputs, rejections and accepted records.
- Starting student checkpoint, tokenizer and adapter configuration.
- Relevant library/runtime versions and hardware.
- Evaluation questions, reference labels, retrieved context and scoring versions.
- Spending records, errors and decisions to stop or continue.

Random seeds help control some randomness. They do not guarantee identical results across hardware, nondeterministic GPU operations, library changes or updated hosted models. Keep the actual outputs and hashes instead of relying on a recipe alone.

A **hash** identifies particular bytes. A **manifest** records artifact identities and relationships. Neither proves semantic correctness. A reproducible wrong dataset is still wrong, but at least it can be diagnosed and replaced precisely.

## 4. Provenance, permission and truth are separate

**Provenance** describes where a record came from and how it was transformed. It lets a reviewer trace a generated citation back to a passage and a source version.

A source may be traceable yet misinterpreted. It may be publicly accessible yet require a separate decision about processing or reuse. It may be technically verified yet unsuitable for a different deployment purpose.

In this project, selected-source machine checks, actual operator consent for research-only Azure processing, and independent legal clearance are distinct. The recorded consent is not a universal license for every later model release or commercial use. Reassess permissions when the purpose, processing location or distribution plan changes.

## 5. Caches and locks protect different things

A **cache** reuses an earlier computation. It is safe only when the inputs and relevant settings match. Reusing answers from an old question set or retrieval results from an old index can invalidate comparisons.

This project distinguishes the Phase 1 search index from the later shared evaluation retrieval cache. The first enables search; the second can preserve identical evidence inputs across model comparisons. They are not interchangeable artifacts.

A **publication lock** prevents competing jobs from publishing into the same output location. A **completion manifest** indicates which required artifacts were produced and verified. A directory containing some files is not equivalent to a completed run.

After failure, preserve evidence before repairing state. Deleting a spending ledger or an execution marker to make a paid run look new can erase budget accountability and trigger duplicate work.

## 6. Why the bounded pilot exists

The selected pilot requests **300 training examples and 50 evaluation questions**, with a **$50 local ledger ceiling**, one worker, one attempt and no automatic refill. A request count is not an accepted count. Rejections, insufficient eligible source material or service failures can leave fewer usable records.

The local ledger is a workflow spending control, not an independent guarantee about the provider's final bill. Tokens, failed or ambiguous requests, price changes and retries must be understood under the provider's actual accounting rules.

The full target—3,650 accepted training examples and 1,100 evaluation questions—is a separate experiment. It requires its own review, authorization and compute planning. A successful source/index preflight does not authorize paid generation, and a successful generation pilot does not prove fine-tuning will help.

The pilot is valuable when it answers concrete questions cheaply: Are the prompts producing usable records? Which slices fail? How much review is needed? What is the observed cost per accepted item? Fix systemic defects before scaling them.

## 7. Process survival is not experiment recovery

The remote preparation job was moved into **tmux** so it can survive an SSH or laptop disconnect. This is an operational convenience, not an ML technique. It does not survive VM shutdown by itself, validate outputs or make partial embeddings resumable.

A durable pipeline needs explicit completion checks, preserved failure evidence and stage-aware recovery. Do not infer readiness from an old progress message. Current operational status belongs in the status report and live run artifacts, not in this conceptual guide.

## 8. Deployment starts a feedback loop

After release, monitor error types, source freshness, retrieval failures, latency, cost and user review burden. Changes in the input population are **distribution drift**. Changes in the relationship between input and correct answer—such as a revised policy—can be **concept drift**.

New user feedback is candidate evidence, not automatically a trusted label. Review it, protect sensitive information, and decide whether the right fix is a source update, prompt change, retrieval repair, new examples or retraining.

Keep the last acceptable model, its compatible index and configuration available for rollback. Update one component deliberately, evaluate, and preserve its lineage.

## Alternatives worth recognizing

- **Managed experiment tracking:** centralizes metrics/artifacts; adds service and governance decisions.
- **Versioned files plus manifests:** simpler and portable; requires disciplined conventions.
- **Scheduled retraining:** predictable cadence, but may waste resources when nothing meaningful changed.
- **Performance-triggered updates:** respond to measured drift or failure, provided monitoring is trustworthy.

## Self-check

1. Does a passing preflight prove the generated answers will be correct?
2. Why keep rejected candidate examples?
3. Does tmux make a run recoverable after every kind of failure?

**Answers:**

1. No. It checks specified operational contracts; semantic quality requires review and evaluation.
2. They reveal systematic failure patterns and explain acceptance yield and spending. Retention must still follow the project's data rules.
3. No. It preserves the remote session across disconnection, not arbitrary machine failures or intermediate computation.

**Professional phrasing:** “We version the data, model, retrieval and evaluation artifacts together, and treat operational readiness and measured model quality as separate release gates.”
