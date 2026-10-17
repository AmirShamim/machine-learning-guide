# Module 10 — Designing your next ML project

[Guide home](README.md) · [Previous: lifecycle](09-lifecycle-and-reproducibility.md) · [Next: glossary](11-glossary.md)

## What you should understand

- How to choose a method from the problem, rather than starting with “we need fine-tuning.”
- How to diagnose quality problems before spending on more data or a larger model.
- How to write a small, defensible experiment plan.
- Which knowledge transfers from this project and which settings must be reconsidered.

## 1. Begin with a decision, not a model name

For a new support assistant, “build a smarter chatbot” is too vague. A more useful task description is:

> Given a customer's request, approved product-policy evidence and available transaction details, produce a supported explanation and an eligibility recommendation, or identify the missing information. A person remains responsible for approval during the pilot.

This defines the input, output, evidence boundary and deployment role. Next define what the assistant must not do: invent receipt details, use an obsolete policy or present missing evidence as proof of ineligibility.

Do the same for a classification, prediction or recommendation project. Ask what action the output supports and how mistakes will be detected. Some tasks do not need a language model at all.

## 2. Use the smallest intervention that addresses the failure

| Need or observed problem | Reasonable first approach | Why it may not be enough |
|---|---|---|
| Exact calculation or deterministic eligibility rule | Ordinary code or a rules engine | Unstructured evidence may first need reliable extraction |
| General model can do the task but instructions are vague | Better prompts and a few demonstrations | Context limits and inconsistent behavior may remain |
| Relevant knowledge is private, versioned or frequently changing | RAG over approved evidence | Retrieval and evidence use can still fail |
| Persistent output-format or task-behavior weakness despite good context | SFT with representative, verified examples | Can overfit or teach errors; needs comparison against simpler baselines |
| Need a smaller/cheaper model with a stronger model's task behavior | Distillation and appropriate adaptation | Student capacity and teacher quality limit transfer |
| Need to act on external systems | Controlled tools and application logic | Fine-tuning is not a permissions or transaction-control mechanism |

These methods combine. For example, use a model to extract transaction fields, validate those fields, apply a deterministic eligibility rule, then ask the model to explain the rule's result with evidence. Do not ask probabilistic text generation to replace arithmetic just because a model is available.

## 3. Build an evaluation set before a large training set

Collect representative cases from the intended use, with appropriate permission. Include common requests, important rare cases, ambiguous requests, missing evidence and source-version changes. Establish the expected outcome and what evidence supports it.

Keep evaluation material separate from training. If you repeatedly use a set to redesign prompts or select adapters, it is serving as a development set. Preserve a further held-out evaluation for your final claim.

A narrow high-quality benchmark can be more useful than thousands of automatically labelled questions whose answers nobody trusts. Start with enough varied cases to find large failures; expand the set to support stronger claims and narrower uncertainty.

Our project follows this principle through separate source regions and separate evaluation generation, but these mechanisms do not eliminate every contamination risk. Teacher-generated questions still require checks, and public sources may have appeared in the starting model's training.

## 4. Diagnose before increasing scale

| Symptom | Inspect first | Avoid assuming |
|---|---|---|
| Correct information never reaches the model | Corpus coverage, permissions, filtering and retrieval | More LoRA epochs will repair missing evidence |
| Correct passage is present, but a condition is ignored | Prompt, task difficulty and context-grounding targets | The embedding model must be wrong |
| Training loss falls while validation/task quality worsens | Overfitting, split leakage and label consistency | More examples of the same flawed pattern will fix it |
| Citations exist but do not support conclusions | Evidence-to-claim alignment and citation supervision | Citation formatting is equivalent to grounding |
| Model refuses almost everything | Label balance, prompts and no-support examples | It has become reliably cautious |
| Strong pilot score collapses on a new source | Distribution shift, source diversity and group splits | The pilot proved general expertise |
| Local export differs from benchmark results | Prompt template, tokenizer, precision and runtime | Fine-tuning itself explains every difference |

Treat each diagnosis as a hypothesis. Change a component, preserve the old version and compare on appropriate cases. Changing the prompts, data mix, retriever, checkpoint and sampling settings simultaneously makes attribution difficult.

## 5. A practical experiment brief

Before paying for large generation or training, write a compact brief with these fields:

1. **Task and deployment role:** who uses the output, for what action, with what supervision?
2. **Population and scope:** which inputs, languages, source versions and excluded cases?
3. **Baseline:** what can an unchanged model, a prompt or ordinary code already do?
4. **Hypothesis:** what specific failure should the proposed intervention reduce?
5. **Data:** where do inputs and targets come from, and what review/permissions apply?
6. **Split:** what unit is held out—examples, customers, documents, organizations or time?
7. **Measurements:** main quality metric, important category checks, cost and review effort.
8. **Acceptance rule:** what result warrants continuing, and what result means stop?
9. **Budget and recovery:** spending cap, retries, accepted yield and preserved evidence.
10. **Release and rollback:** who approves, which artifact versions ship, and how to revert?

A useful support-assistant hypothesis might be: “Evidence-grounding examples reduce unsupported approvals relative to the same base model with the same RAG context, without materially increasing missed eligible requests.” Decide what “materially” means before evaluating; the tolerance depends on business and review costs.

## 6. The human role becomes smaller and more deliberate—not zero

AI can draft prompts, suggest labels, create candidate examples, explain failures and help implement the pipeline. Those are productivity tools, not independent proof of correctness.

Reserve human attention for choices that define the system: the task, acceptable errors, trusted reference labels, source/use decisions and final acceptance. Use automated checks and disagreement-driven sampling to make that review focused.

A solo developer can build and test a prototype. A dependable system in a specialist or consequential setting may require domain expertise and independent review. Distinguishing a prototype from a validated product is professional engineering, not an admission that the prototype was worthless.

For this project, “the code runs” and “v2 is better than v1” are different milestones. The first enables the experiment; the second requires measured evidence.

## 7. What to reuse from this project

Reuse the **reasoning pattern**: scoped sources, traceable examples, held-out evaluation, clear baselines, bounded pilots and explicit release gates.

Do not automatically reuse its 7B checkpoint, context limit, LoRA rank, two epochs, retrieval weights, historical source scope or training-slice proportions. Those are choices for particular constraints. New tasks require renewed evidence that they are appropriate.

Study advanced methods when a diagnosed problem calls for them. You do not need to master every branch of ML before running a careful supervised-learning experiment.

## Alternatives worth recognizing

- **Classical supervised models:** logistic regression or tree models can be excellent for structured prediction.
- **Active learning:** prioritize examples whose labels would most improve the model or clarify uncertainty.
- **Preference optimization:** train from better/worse response comparisons; preferences still need a reliable definition.
- **No ML:** often best when requirements are stable, inputs structured and the desired logic explicit.

## Self-check and design exercise

1. A policy changes weekly. Should you automatically retrain weekly?
2. A fine-tuned model beats plain Base but not Base + RAG. What claim remains justified?
3. Sketch a new project brief using a task you understand. Which independent evidence would convince you not to deploy it?

**Answers:**

1. No. A versioned evidence store and RAG may address the knowledge change. Evaluate before deciding whether behavior also needs retraining.
2. Only the comparisons actually measured. The evidence may favor retrieval without adaptation; it does not establish an added benefit from fine-tuning under RAG.
3. There is no single brief. A useful answer names a task, baseline, held-out cases, measurable error costs and a stopping rule. Examples of contrary evidence include unacceptable false positives, poor performance on a key category, or no improvement over a cheaper baseline.

**Professional phrasing:** “The next step is not automatically a larger model; it is an experiment that isolates the limiting component and tests the cheapest credible intervention.”
