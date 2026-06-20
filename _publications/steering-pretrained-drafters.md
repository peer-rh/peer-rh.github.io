---
title: "Steering Pretrained Drafters During Speculative Decoding"
authors: "Frédéric Berdoz, Peer Rheinboldt, and Roger Wattenhofer"
venue: "Proceedings of the AAAI Conference on Artificial Intelligence"
year: 2026
date: 2026-03-14
github: "https://github.com/ETH-DISCO/SD-square"
arxiv: "https://arxiv.org/abs/2511.09844"
paper: "https://ojs.aaai.org/index.php/AAAI/article/view/40255"
bibtex: "/assets/files/steering-pretrained-drafters.bib"
description: "A lightweight steering mechanism improves pretrained drafter alignment for speculative decoding with negligible computational overhead."
abstract: "Speculative decoding accelerates language model inference by separating generation into fast drafting and parallel verification. Its main limitation is drafter-verifier misalignment, which limits token acceptance and reduces overall effectiveness. While small drafting heads trained from scratch compensate with speed, they struggle when verification dominates latency or when inputs are out of distribution. In contrast, pretrained drafters, though slower, achieve higher acceptance rates thanks to stronger standalone generation capabilities, making them competitive when drafting latency is negligible relative to verification or communication overhead. In this work, we aim to improve the acceptance rates of pretrained drafters by introducing a lightweight dynamic alignment mechanism: a steering vector computed from the verifier's hidden states and injected into the pretrained drafter. Compared to existing offline alignment methods such as distillation, our approach boosts the number of accepted tokens by up to 35% under standard sampling and 22% under greedy sampling, all while incurring negligible computational overhead. Importantly, our approach can be retrofitted to existing architectures and pretrained models, enabling rapid adoption."
doi: "10.1609/aaai.v40i36.40255"
---

## Overview

<figure>
<img src="/assets/publications/steering-pretrained-drafters/method-overview.png" alt="Overview of different drafting paradigms: independent, dependent (EAGLE-3), and SD2, which steers a pretrained drafter using verifier hidden states.">
<figcaption>
<strong>Figure 1:</strong> Overview of different drafting paradigms. Independent drafting uses a smaller model with no access to the verifier's internal state. Dependent drafting (e.g., EAGLE-3) uses lightweight heads trained to read the verifier's hidden states. SD<sup>2</sup> strikes a middle ground, leveraging verifier features for steering while retaining the generalization capabilities of independent drafters.
</figcaption>
</figure>

Speculative decoding accelerates LLM inference by having a lightweight drafter propose tokens that are then verified in parallel by the larger model. The main bottleneck is drafter-verifier misalignment: the more the drafter's predictions diverge from the verifier's, the more tokens get rejected.

Pretrained drafters generalize well but lack dynamic alignment with the verifier. Dependent drafters (trained from scratch on verifier hidden states) are fast but brittle outside their training distribution. SD<sup>2</sup> bridges this gap by introducing a lightweight steering mechanism that extracts predictive signals from the verifier's hidden states and injects them into the pretrained drafter at inference time.

### How it works

<figure>
<img src="/assets/publications/steering-pretrained-drafters/detail-method.png" alt="Diagram of the SD2 steering mechanism showing how verifier hidden features are projected into biases added to the drafter's MLP layers.">
<figcaption>
<strong>Figure 2:</strong> The steering mechanism concatenates the verifier's high-, medium-, and low-level hidden features and passes them through a linear projection to produce a steering vector. This vector is transformed into a set of biases added to all MLP hidden states in the drafter, just before the activation function.
</figcaption>
</figure>

The method has three components:

1. **Steering vector extraction:** After verification, the verifier's hidden states at the last accepted position are concatenated across three layers (high, mid, low) and linearly projected into a compact steering vector.
2. **Bias injection:** The steering vector is mapped through a second linear layer into per-layer biases, which are added inside the drafter's SwiGLU MLP (before the gating activation) at every layer.
3. **Training:** The steering parameters and the drafter are jointly fine-tuned using KL divergence against the verifier's token distribution, with a random offset to simulate all drafting positions uniformly.

<figure>
<img src="/assets/publications/steering-pretrained-drafters/training.png" alt="Training procedure of SD2 showing how the drafter is aligned to the verifier using KL divergence with a random drafting offset.">
<figcaption>
<strong>Figure 3:</strong> Training aligns the steered drafter's distribution to the verifier's. A random offset simulates different drafting positions so the steering mechanism learns to encode information about the next k tokens.
</figcaption>
</figure>

## Results

SD<sup>2</sup> is evaluated across four verifier-drafter pairs (Vicuna 1.3 13B / Llama 160M, Qwen3 14B / Qwen3 0.6B, Qwen3 8B / Qwen3 0.6B, Llama 3.1 8B / Llama 3.2 1B) on five benchmarks (UltraChat, HumanEval, XSum, Alpaca, GSM8K).

<figure>
<img src="/assets/publications/steering-pretrained-drafters/n_accepted.png" alt="Bar chart showing average accepted tokens per block for EAGLE-3, pretrained, distilled, and SD2 across model pairs.">
<figcaption>
<strong>Figure 4:</strong> Average number of accepted tokens per block across all tasks. Solid bars: T=1 (sampling); hatched bars: T=0 (greedy). SD<sup>2</sup> consistently achieves higher acceptance rates than both pretrained and distilled drafters. For Vicuna 1.3 with the small Llama 160M drafter, SD<sup>2</sup> closes the gap with the dependent EAGLE-3 head.
</figcaption>
</figure>

### Key numbers

- **Up to +35% more accepted tokens** under standard sampling and +22% under greedy sampling compared to pretrained drafters.
- **Up to +83% speedup** on in-distribution data (Vicuna 1.3), and +61% average speedup across all tasks for that pair.
- SD<sup>2</sup> consistently outperforms distillation, achieving roughly **twice the additive speedup** under standard sampling.
- On out-of-distribution tasks (GSM8K, HumanEval), SD<sup>2</sup> matches or exceeds pretrained drafter performance, while distillation often degrades.

### Long-sequence robustness

<figure>
<img src="/assets/publications/steering-pretrained-drafters/bucket_comparison.png" alt="Line plots showing accepted tokens per block at different sequence positions for all model pairs.">
<figcaption>
<strong>Figure 5:</strong> Accepted tokens per block at different positions in the generated sequence. SD<sup>2</sup> maintains a consistent advantage over distillation across all token positions, confirming that steering works well with increasing sequence length.
</figcaption>
</figure>

### Ablations

<figure>
<img src="/assets/publications/steering-pretrained-drafters/ablation_all_but_offset.png" alt="Ablation study showing the effect of different steering mechanisms and drafter freezing on block efficiency.">
<figcaption>
<strong>Figure 6:</strong> Ablation on Vicuna 1.3 7B / Llama 160M. SD<sup>2</sup>'s design (bias in MLP before gating + unfrozen drafter) consistently outperforms alternatives: post-MLP bias, conditional bias, and frozen-drafter variants. Steering alone (frozen drafter) still doubles accepted tokens over the pretrained baseline.
</figcaption>
</figure>

The ablation studies justify two key design choices:

- **Steering mechanism:** Adding the bias inside the SwiGLU (before gating) outperforms simpler post-MLP injection and more complex conditional variants.
- **Drafter fine-tuning:** Unfreezing the drafter during training yields consistently better results, though steering alone already doubles accepted tokens over the pretrained baseline.

<div class="callout">
<strong>Key takeaway:</strong> A lightweight steering vector extracted from the verifier's hidden states and injected as a bias into the drafter's MLP layers can substantially improve speculative decoding acceptance rates, with negligible computational overhead and full compatibility with existing pretrained drafter-verifier pairs.
</div>

## Code

The implementation is available on GitHub: [ETH-DISCO/SD-square](https://github.com/ETH-DISCO/SD-square).

## Citation

Berdoz, F., Rheinboldt, P., & Wattenhofer, R. (2026). Steering Pretrained Drafters During Speculative Decoding. <em>Proceedings of the AAAI Conference on Artificial Intelligence, 40</em>(36), 30067-30075. <a href="https://doi.org/10.1609/aaai.v40i36.40255">https://doi.org/10.1609/aaai.v40i36.40255</a>
