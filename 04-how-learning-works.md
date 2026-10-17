# 04 · How learning works

[Guide home](README.md) · Previous: [Data and supervision](03-data-and-supervision.md) · Next: [Splits, leakage, and generalization](05-splits-leakage-and-generalization.md)

## Learning goals

- Explain loss, gradients, and updates as parts of one training loop.
- Distinguish learning the target text from meeting application objectives.
- Recognize underfitting, overfitting, and when data problems should take priority.

## A training loop is an adjustment loop

Training compares predictions with targets, estimates adjustment directions, and updates trainable parameters to make desired outputs more likely.

Think of tuning configuration values against tests. Unlike ordinary debugging, training uses numerical derivatives rather than human-readable fixes, and its immediate score is not a comprehensive acceptance test.

Humans select objectives, data, settings, and acceptance criteria. They investigate failures and judge suitability, without approving every numerical adjustment.

## Loss is not the product metric

A **loss** is the numerical objective minimized during training. A **metric** measures task outcomes, such as correct dispositions or citations. Loss supports optimization; metrics represent application success. They need not improve together.

For next-token supervision, a common loss is **negative log likelihood**: assigning a higher probability to the desired next token produces a smaller penalty. For one target token:

$$
L = -\ln(p)
$$

Here, $L$ is the loss for that token, $p$ is the probability the model assigns to the desired token given the preceding context, and $\ln$ is the natural logarithm. If $p$ is 0.5, the loss is about 0.69; if $p$ is 0.1, it is about 2.30. The lower value rewards greater probability on the target. These numbers are not probabilities that the complete answer is legally or factually correct.

Training aggregates losses across eligible tokens and examples. It rewards the supplied target sequence, including its wording. Two equally valid phrasings can receive different losses if only one is supplied as the target.

## Worked example: learning the desired response

Use fictional equipment policy version 3, which requires a receipt and permits returns within 30 days. The prompt describes a day-24 request with no receipt. The target response says that the deadline condition is satisfied but receipt evidence is needed.

During supervised training, the model predicts each assistant token using the prompt and the earlier target tokens. This is often called **teacher forcing**. At ordinary generation time, it instead continues from its own generated tokens, so an early mistake can affect later text.

With **assistant-only loss**, prompt tokens do not directly contribute target-token penalties. They still condition the response and influence the computations used to learn. Masking prompt loss does not mean ignoring the prompt.

A mislabeled “approve immediately” target can teach the wrong behavior despite falling loss. This simplified business policy does not substitute for qualified judgment in genuine legal cases.

## What makes an update happen?

A **batch** is a group of examples processed together. An **epoch** is one pass through the training examples. A training **step** commonly means an optimizer update; with gradient accumulation, several smaller batches can contribute to one update. Always check what a reported step actually counts.

A **gradient** describes how small parameter changes would change loss locally. **Backpropagation** computes these derivatives through the network. An **optimizer** uses them to update parameters. These calculations do not judge legal conclusions.

The **learning rate** controls the scale of updates. Too large an update can destabilize learning; too small can make useful change slow. There is no universally correct setting independent of model, data, and training setup.

## Fitting versus generalizing

**Underfitting** means inadequately learning useful patterns, often performing poorly even on training examples. **Overfitting** means fitting details that do not transfer. Falling training error alongside worsening held-out error is a warning sign. Held-out cases are not used for updates; see [Module 05](05-splits-leakage-and-generalization.md).

**Regularization** constrains undesirable fitting, for example by limiting training duration or discouraging large parameter changes. It cannot guarantee preservation of existing capabilities.

**Transfer learning** starts from a model that already learned useful patterns instead of training from scratch. Fine-tuning adapts that starting point to a narrower task. Falling loss alone proves neither reasoning improvement nor better source fidelity.

## How this applies here

V2 intends BF16 LoRA training with a fresh adapter on Qwen/Qwen2.5-7B-Instruct, not continuation from v1. LoRA updates a restricted set of added parameters while the base weights remain frozen. It does not guarantee that the resulting system avoids forgetting or degraded behavior; see [Module 07](07-fine-tuning-lora-and-export.md).

The current indexing work performs retrieval preparation, not these training updates. No fine-tuning success has been established. Before scaling a pilot, review label fidelity, policy or instrument versions, missing evidence, and slice balance. More updates cannot repair a systematically wrong target definition.

## Common misconceptions

- “Lower loss means better reasoning.” It primarily means better target prediction.
- “Frozen base weights prevent every regression.” Adapted outputs can still degrade.
- “More training fixes bad examples.” It can reinforce them.

## Alternative methods

- **Prompting only:** changes request-time instructions without updating weights.
- **RAG:** supplies retrieved evidence without requiring weight adaptation.
- **Full fine-tuning:** updates base parameters, with different resource and regression considerations.

## Self-check

1. Why can loss improve while answer quality worsens?
2. Does assistant-only loss discard the prompt?
3. What should be checked before adding more training?

### Answer key

1. The model may fit incorrect targets or non-transferable details.
2. No; the prompt still conditions assistant-token predictions.
3. Check target correctness, source versions, coverage, and held-out behavior before scaling updates.
