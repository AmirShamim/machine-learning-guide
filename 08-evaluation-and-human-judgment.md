# Module 08 — Evaluation, experiments and human judgment

[Guide home](README.md) · [Previous: adaptation](07-fine-tuning-lora-and-export.md) · [Next: lifecycle](09-lifecycle-and-reproducibility.md)

## What you should understand

- How to turn “more productive than v1” into a defensible experiment.
- Why a single score can conceal important failures.
- What AI judges can contribute without becoming unquestionable authorities.
- Where human work has the highest leverage.

## 1. A model claim needs a baseline and a measurement

“More professional-looking answers” and “more correct answers” are different properties. Longer explanations may be more fluent but slower to review. A productive system might improve accuracy, reduce review time, lower false positives, or reduce cost while preserving an agreed quality threshold.

Choose the claim before reading the results. Otherwise it is easy to select whichever metric happened to improve and describe the experiment as a success.

A **baseline** is a reference system. In this project the unchanged Qwen checkpoint and historical v1 are relevant baselines, with compatibility and exposure limitations documented. A simpler prompt-only or retrieval-only system can also be a valuable engineering baseline. Complexity has to justify itself.

## 2. Separate measurable dimensions

| Dimension | Question it answers | Important limit |
|---|---|---|
| Decision correctness | Was the requested classification or determination right? | Depends on trustworthy reference labels |
| Evidence grounding | Does the supplied evidence support the answer? | Real citations alone are insufficient |
| Retrieval quality | Did the needed evidence reach the answer model? | Finding a passage does not mean using it correctly |
| Format compliance | Did the answer satisfy the required structure? | Easy to measure, but weak evidence of substantive correctness |
| Robustness | Does performance survive paraphrases, missing evidence and irrelevant context? | A few examples cannot establish universal robustness |
| Operational utility | What are latency, cost and human review effort? | Faster generation may produce slower review |

Do not silently drop failures, refusals or invalid outputs and then report accuracy only on the easy remainder. Record coverage and the reasons for missing outputs. A system that answers only ten of one hundred requests needs that limitation stated.

## 3. A tiny metric example

Suppose a **fictional binary support-policy test** contains 100 requests: 40 eligible and 60 ineligible. The model correctly approves 36, wrongly approves 12, misses 4 eligible requests and correctly rejects 48.

- **True positives:** 36 correct approvals.
- **False positives:** 12 incorrect approvals.
- **False negatives:** 4 missed eligible requests.
- **True negatives:** 48 correct rejections.
- **Accuracy:** 84 correct decisions out of 100, or 84%.
- **Precision for approval:** 36 correct approvals out of 48 approvals, or 75%.
- **Recall for approval:** 36 found eligible requests out of 40 eligible requests, or 90%.

An “always reject” system scores 60% accuracy here while helping no eligible requester. Class balance matters. Precision and recall reveal different failure costs.

The real project is not just this binary task: it distinguishes findings from insufficient evidence and evaluates generated explanations. Do not force missing-evidence cases into a binary true/false label merely to simplify the metric.

For retrieval, if three passages are designated relevant and two are retrieved, passage recall is two-thirds. **Recall@k** limits the retrieved set to its top $k$ results. The value $k$ is the retrieval cutoff, not a probability. Averages across questions should be described explicitly; questions without gold evidence need separate treatment.

## 4. The six-way comparison isolates useful questions

The intended full benchmark uses the same question set across three models and two input regimes:

| Model | Plain prompt | Prompt plus retrieved evidence |
|---|---|---|
| Unchanged Qwen baseline | Base | Base + RAG |
| Historical v1 | v1 | v1 + RAG |
| New v2 | v2 | v2 + RAG |

This supports several comparisons:

- **v2 vs Base:** what adaptation adds without retrieval.
- **Base + RAG vs Base:** what retrieval adds to the unchanged model.
- **v2 + RAG vs Base + RAG:** whether adaptation adds value when both have evidence.
- **v2 vs v1 under the same regime:** whether the new approach improves the measured tasks.

An **ablation** removes or changes a component to estimate its contribution. Keep other conditions as comparable as possible: questions, retrieved evidence, context limits, precision, sampling and scoring. If multiple things change, avoid attributing the whole difference to one of them.

The full target is **1,100 questions and 6,600 responses**, not a completed benchmark. The current 300-training/50-evaluation dataset pilot tests generation and review readiness; it does not automatically train v2 or execute those six model runs.

## 5. AI judges are instruments that need checking

Some checks are deterministic: required fields, valid option labels, known citation identifiers and exact quote matching. Other checks require semantic judgment: whether a passage supports a conclusion or whether important conditions were omitted.

A capable model can propose scores and explanations. But model judgments can be affected by style, verbosity, order, shared blind spots and the same misconceptions as the generator. Written rationales help inspection; they are not proof that a judgment is right or a transcript of hidden internal reasoning.

The design therefore calls for calibration against independently labelled examples, independent-family second/meta judgments, repeated cases and canaries. **Judge calibration** here means checking a grading instrument against references; it is not automatically statistical calibration of the answer model's confidence probabilities.

Sol and Luna are configured as same-family deployments. They are not independent merely because their names differ. Having a second AI agree with the first is weaker evidence when their errors are correlated.

## 6. What humans need to do

Humans need not manually type every training example. High-leverage responsibilities include:

1. Define the intended task, source scope and acceptable failure costs.
2. Establish or verify trusted reference labels and difficult boundary cases.
3. Review generated examples across categories, sources and failure types.
4. Inspect disagreements between rules, models and reviewers.
5. Decide whether observed quality supports the proposed use and level of supervision.

A developer can verify structure and reproducibility. Qualified domain reviewers are needed where the interpretation itself requires expertise. A random sample of 50 examples can reveal obvious defects; it cannot certify every example or replace independent evaluation.

**Risk-based sampling** prioritizes high-impact cases. **Stratified sampling** ensures small but important categories appear in a review. **Disagreement sampling** directs attention to examples where reviewers or models differ. These reduce review waste, not the need to set meaningful acceptance criteria.

## 7. Report uncertainty, not just a winner

Observed scores vary with the sample. A tiny average improvement may not be stable. Paired comparisons examine differences on the same questions; confidence intervals describe sampling uncertainty under their assumptions. A paired bootstrap is one possible method, but related questions from the same document require care because they are not fully independent.

Report category-level failures, missing outputs, data scope, runtime conditions and review limitations. A null or negative result is still useful: it can show that retrieval already supplies most of the value, that supervision is weak, or that the proposed adaptation is not worth its cost.

## Alternatives worth recognizing

- **Expert-only grading:** strong domain accountability, with higher cost and possible reviewer disagreement.
- **Model-assisted review:** scalable triage with targeted human adjudication; requires judge validation.
- **Deterministic task benchmarks:** reliable for objectively checkable outputs, narrower than open-ended quality.
- **Shadow deployment:** evaluate on real traffic without acting on outputs; useful only with appropriate permissions and controls.

## Self-check

1. Does a higher format-compliance score prove better substantive analysis?
2. Why compare v2 + RAG against Base + RAG, not only against plain Base?
3. Can two same-family AI judges replace an independent reference standard?

**Answers:**

1. No. Formatting is one dimension and can improve while correctness worsens.
2. Otherwise the improvement could come from evidence access rather than adaptation. Equal retrieval conditions help isolate the contribution of fine-tuning.
3. No. Agreement can reflect shared errors. Use independently established labels and qualified review where required.

**Professional phrasing:** “We will accept the adaptation only if paired evaluation shows useful task-level gains under comparable conditions, without unacceptable category-level regressions.”
