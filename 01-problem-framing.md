# 01 · Problem framing: decide what success means

[Guide home](README.md) · Previous: [Introduction](README.md) · Next: [Models, tokens, and inference](02-models-tokens-and-inference.md)

## Learning goals

- Distinguish learned behavior from explicit rules.
- Specify inputs, outputs, and the population the system should serve.
- Choose measurable objectives, a baseline, and meaningful error costs.

## Start with a decision, not a model

Machine learning (ML) learns patterns from examples rather than requiring a developer to specify every decision rule. A rules engine follows conditions you write: if a purchase is more than 30 days old, apply the stated deadline rule. An ML model estimates a useful output from patterns learned during training. Neither approach automatically understands your product requirements.

A developer analogy is implementing a function from examples instead of writing its complete body. The examples constrain the behavior, but they do not provide a formal specification or guarantee that unseen inputs work. Unlike a compiler, training does not prove that the resulting function satisfies your intended contract.

Use rules for explicit requirements and ML for variable inputs, such as differently worded customer messages. A hybrid can extract facts with ML and apply deterministic rules, but extraction errors remain possible.

## Define the input-output contract

An input-output contract specifies available information, required outputs, and failure behavior. Inputs might include the message, purchase date, receipt status, policy version, and supporting passages. Outputs could include a recommendation, references, and missing information. “Answer helpfully” is not testable.

**Features** are the information used to predict an output. In a conventional model they might be named fields such as days since purchase. A language model also builds internal representations from text; a developer need not manually enumerate every linguistic feature. The **target** is the desired output used for supervision or assessment, not simply whatever the model happens to produce.

Three common task types differ:

- **Classification:** choose a category, such as eligible, ineligible, or insufficient evidence.
- **Regression:** predict a numerical quantity, such as expected handling time. Computing an exact date difference is arithmetic, not necessarily regression.
- **Generation:** produce a sequence of text, such as an explanation grounded in a policy.

An application combining these tasks needs separate checks for each.

## Worked example: an equipment return

Consider a fictional equipment support policy, version 3, effective for this purchase. It allows returns within 30 days and requires receipt evidence. A customer contacts support 24 days after purchase, but no receipt is supplied.

The correct response under this simplified policy is not an unconditional approval. The time condition is satisfied, but required evidence is missing. A useful answer states that distinction, asks for the receipt, and identifies version 3 as its source. It should not invent an exception or assume that a previous policy version applies.

This is a benign business-policy exercise, not a legal example. Genuine legal interpretation involves additional authority, context, and qualified judgment that this simplified contract cannot represent.

## Measure usefulness, not confident wording

The **population** is the set of cases you intend to handle: perhaps equipment return requests under specified policy versions. The **distribution** describes how common different inputs are within that population. A dataset dominated by complete receipts will not adequately represent a service receiving many incomplete requests. A later policy change can shift that distribution.

A measurable objective could combine correct dispositions, correct source versions, and appropriate requests for missing evidence. Define “productive” through measured outcomes—such as fewer unsupported approvals without excessive unnecessary escalations—not confident language or longer answers.

For a binary approval decision, a **false positive** approves a case that should not be approved; a **false negative** rejects one that should be approved. Their costs may differ. An explicit insufficient-evidence category should be measured too, rather than silently treating every deferral as a rejection. Set acceptable thresholds before comparing systems.

A **baseline** is a reference system, such as a checklist plus keyword search. Compare on identical cases, not natural-sounding responses.

## How this applies here

This project starts from Qwen/Qwen2.5-7B-Instruct and aims to improve source-grounded responses in a legal domain. There is no verified finding yet that v2 is better. Fluent answers remain uncertain outputs, not certificates of correctness.

The selected materials include seven official UN/ICTY records and nine historical legal instruments. Their scope includes the original Rome text corrected through 1999 and original 2002 Rules/Elements; it is not a complete current-law collection. Qualified reviewers must judge legal meaning and acceptance criteria. Machine-verified provenance establishes source checks, not legal correctness or operator processing consent. Humans define objectives and review representative cases; they need not hand-label every item.

## Common misconceptions

- “ML replaces specifications.” It makes an explicit specification more important.
- “High confidence means correct.” Fluent certainty can accompany an unsupported answer.
- “One accuracy score is enough.” Different mistakes can carry different costs.

## Alternative methods

- **Rules and forms:** strong for explicit requirements and structured inputs.
- **Search with human review:** retrieves evidence without generating conclusions.
- **Small classifiers:** useful when the required output is a bounded category.

## Self-check

1. Why is calculating days since purchase not necessarily ML?
2. What should happen when the receipt is missing?
3. Why can verified provenance not establish legal correctness?

### Answer key

1. Exact arithmetic needs no learned prediction.
2. Report the satisfied deadline condition and request evidence; do not assume eligibility.
3. Provenance checks origin, while correctness requires interpreting the relevant authority and facts.
