# Module 07 — Transfer learning, LoRA, precision and export

[Guide home](README.md) · [Previous: retrieval](06-embeddings-rag-and-raft.md) · [Next: evaluation](08-evaluation-and-human-judgment.md)

## What you should understand

- Why adapting an existing model is different from pretraining one.
- What supervised fine-tuning and LoRA change.
- Why quantization, adapter merging and format conversion are different operations.
- Why a smaller adaptation can still overfit or make an existing capability worse.

## 1. Start with capabilities already learned

**Transfer learning** reuses a model trained on one large collection of tasks or data, then adapts it for another purpose. You do not have to teach every language pattern from zero to build a useful specialist system.

This project starts from **Qwen/Qwen2.5-7B-Instruct**, an existing instruction-tuned checkpoint. When the experiment calls it the “base” model, that means the unchanged comparison baseline. It does **not** mean a randomly initialized model or necessarily a raw pretraining-only checkpoint.

**Supervised fine-tuning (SFT)** further trains the model on example inputs and desired outputs. Our examples aim to teach evidence-conditioned responses, appropriate determinations, faithful citations and several output formats. The target is response behavior on a defined task—not a promise to insert a perfectly reliable legal database into the weights.

AI-generated targets can help scale this work. But training makes the student more likely to produce patterns in those targets, including their errors. The quality of the supervision remains central.

## 2. Full fine-tuning versus parameter-efficient adaptation

**Full fine-tuning** updates all or most of the model's parameters. A large model then needs storage not only for weights but also for gradients, optimizer state and intermediate activations. Training memory is substantially different from the memory required just to hold a model file.

**Parameter-efficient fine-tuning (PEFT)** updates a smaller set of parameters while retaining most of the pretrained model. **LoRA**, low-rank adaptation, is the PEFT method intended here.

A useful developer analogy is a separately versioned modification layered on a large dependency. But the modification is learned numerical behavior, not a human-readable patch or a set of explicit business rules. It can affect many outputs beyond the examples you had in mind.

## 3. What LoRA actually represents

For a selected weight matrix, LoRA learns a low-rank change:

$$
W_{\text{adapted}} = W_{\text{original}} + sBA
$$

Here:

- $W_{\text{original}}$ is an existing matrix of model weights, kept frozen during LoRA training.
- $A$ and $B$ are smaller trainable matrices.
- $s$ scales the learned update.
- If $W$ has dimensions $d_{out} \times d_{in}$, then $B$ can have dimensions $d_{out} \times r$ and $A$ dimensions $r \times d_{in}$.
- The **rank** $r$ controls the size of the update representation and is generally much smaller than those original dimensions.

Instead of training $d_{out}d_{in}$ entries for that matrix, the adapter trains approximately $r(d_{out}+d_{in})$ entries. This is one reason adaptation can be cheaper. It does not make activation memory or all training costs disappear.

Rank, the targeted layers and the learning rate are **hyperparameters**: settings selected by the developer rather than learned directly as ordinary weights. Larger rank offers more adaptation capacity, not a guarantee of better generalization.

## 4. How this project uses the method

The intended v2 training uses **bf16 LoRA**, assistant-only target loss, an 8,192-token context limit, two epochs and a small validation split. These are configured choices that still need actual supported-hardware execution and evaluation.

Assistant-only loss means the desired assistant answer contributes training targets, while the prompt supplies conditioning context. The model still processes the prompt, and the learned response depends on it; “no loss on prompt tokens” does not mean the prompt is ignored.

v2 starts a **fresh adapter from the selected Qwen checkpoint**, not from the v1 adapter. This avoids directly inheriting v1's adaptation and makes the comparison easier to interpret. It does not erase knowledge from Qwen's original training or prove no overlap with evaluation sources.

A lower validation loss is encouraging optimization evidence, but the product objective includes correctness, grounding and appropriate uncertainty. Those need task evaluation. A model can predict polished target wording better while still making consequential task mistakes.

Specialization can also degrade previous capabilities, sometimes called **catastrophic forgetting**. LoRA reduces the number of trainable parameters, not this risk to zero. The current dataset design does not include a general-capability replay mixture: it does not deliberately rehearse a broad collection of unrelated tasks during adaptation. Measure the general capabilities you still need; retaining an unchanged base checkpoint is a rollback option, not evidence that the adapted model preserved them.

## 5. Precision is a storage/computation choice

**bf16** is a 16-bit numerical format. It is not full-precision float32. Reduced precision can lower memory and computation costs, with behavior dependent on hardware and implementation.

**Quantization** represents weights using fewer bits, often with scaling information. Lower precision can make a model practical on smaller devices, but introduces approximation. Its effect on quality must be measured.

**QLoRA** combines a quantized frozen backbone with trainable LoRA adapters. It does not mean “train every parameter in four-bit precision.” The historical v1 used QLoRA; the v2 design intentionally uses bf16 LoRA on a suitable GPU.

LoRA answers **which parameters are adapted**. Quantization answers **how numbers are represented**. These are separate dimensions of the design.

## 6. Merge and export happen after learning

An **adapter checkpoint** stores learned adapter parameters and the metadata needed to use them with the correct base. An adapter alone is not usually a complete standalone language model.

**Merging** applies the learned update to the matching base weights to produce a combined model. It does not generate examples, retrieve sources or train another epoch. Applying an adapter to an incompatible checkpoint is not a valid shortcut.

**Exporting** packages or converts the result for a serving environment. The intended outputs include a merged bf16 model and a GGUF Q4_K_M variant for compatible local runtimes. GGUF is a model file format; the quantization variant is a separate property of that file.

A successful file write is not sufficient acceptance. Load the exported model in the target runtime, verify tokenizer/prompt compatibility, and test outputs. Compare systems under consistent precision and runtime settings; do not attribute a change caused by different quantization to fine-tuning alone.

## Alternatives worth recognizing

- **Prompt-only adaptation:** inexpensive first baseline; changes instructions, not weights.
- **Full SFT:** more adaptation freedom, but higher compute/storage cost and still vulnerable to overfitting.
- **QLoRA:** reduces frozen-weight memory for adaptation on constrained hardware.
- **Continued pretraining or preference tuning:** different objectives—domain-text prediction or learning preferences between responses. Neither automatically replaces correct supervised targets or evaluation.

## Self-check

1. Does LoRA guarantee preservation of all original capabilities?
2. Is a GGUF export a new training dataset or a new training run?
3. Why must the adapter identify its original checkpoint?

**Answers:**

1. No. A small learned update can still change behavior broadly or degrade skills. Measure relevant regressions.
2. Neither. It is a deployment artifact derived from model parameters, potentially with quantization.
3. The adapter was learned relative to particular weights and model structure. A similarly named but different base is not automatically compatible.

**Professional phrasing:** “We are doing supervised parameter-efficient adaptation of an existing instruction-tuned checkpoint, followed by verified merging and runtime-specific export.”
