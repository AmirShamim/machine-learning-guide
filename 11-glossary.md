# Module 11 — Glossary and commonly confused concepts

[Guide home](README.md) · [Previous: designing a project](10-designing-your-next-project.md)

Use this as a reference after reading the modules, not as a vocabulary test. Terms are grouped by the architectural question they answer.

## Problem and learning foundations

See [problem framing](01-problem-framing.md), [models and inference](02-models-tokens-and-inference.md) and [learning](04-how-learning-works.md).

| Term | Meaning |
|---|---|
| AI | Broad field of systems performing tasks associated with intelligence; includes approaches that do not learn from data. |
| Machine learning (ML) | Methods that learn useful patterns from data rather than having every decision rule explicitly programmed. |
| Deep learning | ML using neural networks with multiple computational layers. |
| Neural network | A parameterized computation whose numerical parameters are adjusted through learning. |
| Transformer | Neural-network architecture using attention to combine information across input positions; the family underlying Qwen. |
| Attention | Learned, context-dependent weighting and combination of representations. Not a guarantee of human-like understanding or an explanation of the final decision. |
| Parameter / weight | A numerical value learned during training that influences the model's computation. |
| Hyperparameter | A setting chosen outside ordinary parameter learning, such as learning rate, LoRA rank or epoch count. |
| Checkpoint | Saved model state at a particular point, such as pretrained weights or a trained adapter. Some checkpoints also include optimizer state for resuming training. |
| Token | Unit produced by a tokenizer: possibly a word, word fragment, punctuation or another symbol. Not synonymous with a word. |
| Tokenizer | Rules and vocabulary that map text to token IDs and back. Must be compatible with the model. |
| Context window | Maximum token span the configured model/runtime can process for a request. Prompt and generated output share the relevant capacity. |
| Inference | Using a trained model to compute outputs without updating its parameters. |
| Autoregressive generation | Producing successive tokens conditioned on the input and earlier generated tokens. |
| Temperature | Sampling setting that changes the sharpness of next-token probabilities. Lower temperature is not a truth guarantee. |
| Classification | Predicting a category, such as eligible, ineligible or insufficient evidence. |
| Regression | Predicting a numerical value. Distinct from a software regression, which is an unintended loss of previous behavior. |
| Generative task | Producing structured or unstructured content rather than only a predefined label or number. |
| Baseline | Reference system used to determine whether a proposed change adds value. |
| Distribution | The patterns and frequencies of inputs and outcomes in a population. A collected sample approximates it imperfectly. |

## Data and supervision

See [data and supervision](03-data-and-supervision.md) and [splits and leakage](05-splits-leakage-and-generalization.md).

| Term | Meaning |
|---|---|
| Corpus | Collection of source material. Raw sources are not automatically supervised examples. |
| Example | A unit used for learning or evaluation, usually containing an input and a target/reference or label. |
| Label / target | Desired output used as supervision. It can be a category, number or full assistant response. |
| Supervised learning | Learning from examples with target outputs. |
| Self-supervised learning | Constructing a learning signal from the data itself, such as predicting later tokens from earlier ones. |
| Pretraining | Broad initial learning, often from large text collections. This project reuses an existing pretrained model. |
| Instruction tuning | Training on instruction/response examples to improve instruction-following behavior. |
| SFT | Supervised fine-tuning: further training a model against desired outputs. |
| Teacher / student | A teacher supplies supervision or judgments; the student is the model being adapted. Roles do not establish correctness. |
| Distillation | Transferring behavior from a teacher to a student. Here it primarily involves teacher-generated response targets, not necessarily matching teacher probability distributions. |
| Synthetic data | Data generated programmatically or by models. It still needs quality and representativeness checks. |
| Data lineage / provenance | Recorded origins and transformations connecting an artifact to its sources. |
| Training set | Examples used to update parameters. |
| Validation / development set | Examples used to select settings, checkpoints or prompts during development. Repeated consultation makes it part of the development process. |
| Test / held-out evaluation set | Data reserved for measuring the final selected system under a stated protocol. |
| Leakage | Unintended exposure of information that makes evaluation artificially easy or contaminates the development/test boundary. |
| Group split | Keeping related units, such as documents or customers, on one side of a split to reduce dependent overlap. |
| Generalization | Performance on relevant inputs beyond the exact training examples. |
| Distribution shift | A difference between the population represented by development data and the population encountered later. |
| Label noise | Incorrect or inconsistent supervision. |
| Class imbalance | Unequal frequencies of categories. It can hide weak minority-category performance behind high overall accuracy. |
| Data bias | Systematic distortions in selection, coverage or labeling that affect what the model learns and what evaluation measures. |

## Optimization and adaptation

See [how learning works](04-how-learning-works.md) and [fine-tuning and export](07-fine-tuning-lora-and-export.md).

| Term | Meaning |
|---|---|
| Loss | A numerical objective optimized during learning; not necessarily the same as product quality. |
| Cross-entropy / negative log-likelihood | Common objective that penalizes assigning low probability to the target token or label. |
| Gradient | Derivative information describing how a small parameter change affects the loss locally. |
| Backpropagation | Efficient computation of gradients through the model's connected operations. |
| Optimizer | Algorithm using gradients and possibly accumulated state to update parameters. |
| Learning rate | Controls the scale of optimization updates. Too large can destabilize learning; too small may make progress slow or inadequate. |
| Batch | Group of examples or sequences processed together during learning. |
| Gradient accumulation | Combining gradients across smaller batches before updating parameters, often to manage memory. |
| Step | Usually an optimizer update; logging may also use microsteps, so check the specific convention. |
| Epoch | One pass through the training data under the loader's sampling rules. |
| Assistant-only loss | Supervising assistant output tokens while prompt tokens provide context without becoming prediction targets for the loss. |
| Underfitting | Failure to learn useful structure sufficiently, including poor training performance. |
| Overfitting | Learning training-specific details or noise that do not transfer to relevant new inputs. |
| Regularization | Methods or constraints intended to improve generalization by discouraging undesirable fitting behavior. |
| Catastrophic forgetting | Adaptation that substantially degrades previously learned capabilities. |
| Transfer learning | Reusing a learned model or representation as a starting point for a new task. |
| PEFT | Parameter-efficient fine-tuning: adapting a limited set of parameters rather than all weights. |
| LoRA | Low-rank adaptation: learning smaller matrices that define updates to selected frozen weight matrices. |
| Rank | Size of the low-rank update representation; one control on LoRA capacity and parameter count. |
| Adapter | Learned additional parameters used with a compatible base model. |
| bf16 | A 16-bit floating-point format used for supported model computation/storage. |
| Quantization | Lower-bit numerical representation, usually trading some precision for reduced memory or faster compatible execution. |
| QLoRA | LoRA training using a quantized frozen backbone. Not the current intended v2 bf16-LoRA path. |
| Merge | Incorporating a compatible learned adapter update into base model weights. |
| Export | Saving or converting model artifacts for a deployment runtime. |
| GGUF | A model file format used by compatible inference engines; it can hold different numerical representations. |

## Retrieval and evidence

See [embeddings, RAG and RAFT](06-embeddings-rag-and-raft.md).

| Term | Meaning |
|---|---|
| Chunk / passage | A segment of source text. Different stages can use different sizes and boundaries. |
| Embedding | Numerical vector representation of an input. Similarity reflects the representation's learned relationships, not guaranteed truth. |
| Sparse retrieval | Retrieval using sparse features such as lexical terms. BM25 is the method used here. |
| BM25 | A lexical ranking method based on term occurrence, rarity and document-length normalization. |
| Dense retrieval | Retrieval using learned vectors and similarity search. |
| Hybrid retrieval | Combination of lexical and dense retrieval signals. |
| FAISS | Library supporting efficient vector similarity search and clustering. It is not the answer-generating model. |
| Index | Search structure built from a source collection and its representations. |
| Top-k | The highest-ranked $k$ results. Choosing a larger $k$ adds context but can also add noise and cost. |
| Reranker | Second-stage model or scoring process that reorders candidate retrieval results. |
| RAG | Retrieval-augmented generation: supplying retrieved evidence to a model at answer time. |
| RAFT | Retrieval-augmented fine-tuning: training on questions with relevant and distracting context to improve evidence-conditioned behavior. Exact implementations vary. |
| Gold passage | Designated supporting passage for an example; its relevance still needs verification. |
| Distractor | Context not needed for the target answer, used to test or teach selective evidence use. |
| Grounding | Support for generated claims in the permitted evidence. |
| Hallucination | Generated content that is unsupported, fabricated or false in the relevant context. Fluency does not prevent it. |
| Insufficiency / abstention | Reporting that available information does not justify an answer or declining to assert a determination. Not equivalent to proving the negative. |

## Evaluation and operations

See [evaluation](08-evaluation-and-human-judgment.md) and [lifecycle](09-lifecycle-and-reproducibility.md).

| Term | Meaning |
|---|---|
| Metric | Measurement used to summarize a property of system performance. |
| Accuracy | Fraction of scored decisions that match their reference labels under the stated scoring rules. |
| Precision | Among predicted positives, the fraction that are correct. |
| Recall | Among actual positives, the fraction successfully identified. Retrieval recall refers to finding relevant evidence. |
| Coverage | Fraction of required cases with outputs or completed judgments; define which coverage is meant. |
| Slice metric | Performance on a defined subset, such as a source group, task type or no-support category. |
| Ablation | Controlled removal/change of a component to examine its contribution. |
| Paired comparison | Comparing systems on the same cases rather than unrelated samples. |
| Confidence interval | Interval quantifying sampling uncertainty under stated assumptions; not protection against leakage or incorrect reference labels. |
| Judge | Human or model that assesses responses using a rubric. |
| Judge calibration | In this project, testing a candidate judge against independently labelled grades before relying on it. |
| Probability calibration | Alignment between reported predictive probabilities and observed frequencies; distinct from selecting a grading model. |
| Canary | A held-out control case with an independently established expected outcome used to detect evaluation failures. |
| Hash | Fingerprint of particular bytes; establishes identity/integrity, not correctness or permission. |
| Manifest | Record of outputs, input/config identities and completion evidence for an artifact or stage. |
| Preflight | Checks performed before execution to verify specified readiness conditions. |
| Pilot | Bounded experiment intended to test feasibility, quality and cost before scaling. |
| Ledger | Persistent accounting of requests, reservations and spending under the workflow's cost rules. |
| Drift | Change in the incoming data or the relationship between inputs and correct outputs over time. |
| Rollback | Returning to a previously accepted, compatible set of model and system artifacts. |

## The ten distinctions to remember

1. **Corpus ≠ supervised dataset:** reading material is not the same as input/target pairs.
2. **Indexing ≠ training:** search preparation does not update Qwen's weights.
3. **Context ≠ parameters:** retrieved evidence is request input, not automatic permanent learning.
4. **RAG ≠ RAFT:** evidence access at inference differs from supervised practice using evidence.
5. **Loss ≠ task score:** predicting targets better does not guarantee greater utility or correctness.
6. **Validation ≠ final test:** data used for selection is not untouched final evidence.
7. **LoRA ≠ quantization:** parameter adaptation and numerical representation are separate choices.
8. **Merge ≠ dataset generation:** weight combination happens after training.
9. **Passing checks ≠ proven truth:** structural, semantic and operational assurances have different limits.
10. **AI agreement ≠ independent validation:** correlated mistakes can survive multiple model reviews.
