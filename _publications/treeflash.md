---
title: "TreeFlash: Parallel AR-Approximation for Faster Speculative Decoding"
authors: "Peer Rheinboldt, Frédéric Berdoz, and Roger Wattenhofer"
venue: "arXiv preprint"
year: 2026
date: 2026-06-03
github: "https://github.com/ETH-DISCO/TreeFlash"
arxiv: "https://arxiv.org/abs/2606.03819"
paper: "/assets/files/treeflash.pdf"
bibtex: "/assets/files/treeflash.bib"
project:
description: "TreeFlash accelerates tree-based speculative decoding by adding a lightweight autoregressive approximation to one-shot block drafting."
abstract: "One-shot block drafters for speculative decoding generate the full draft in a single forward pass, achieving strong throughput by eliminating sequential token generation. However, they predict each draft token conditioned only on the prefix context, with no dependence on previously drafted tokens. TreeFlash incorporates an MLP layer conditioned on the drafter's hidden state and previous token to approximate an autoregressive distribution while retaining O(1) drafting time complexity."
---

## Overview

<figure>
  <object class="paper-figure" data="/assets/treeflash/method.pdf" type="application/pdf">
    <a href="/assets/treeflash/method.pdf">View Figure 1</a>
  </object>
  <figcaption>
    <strong>Figure 1:</strong> TreeFlash sits between autoregressive and fully marginal one-shot drafting. It keeps the single-pass efficiency of DFlash while adding a lightweight autoregressive approximation layer that conditions each draft position on the preceding drafted token.
  </figcaption>
</figure>

Speculative decoding accelerates LLM inference by letting a cheaper drafter propose multiple future tokens that the target model verifies in parallel. Tree-based speculative decoding improves this further by proposing a tree of possible continuations instead of a single chain.

One-shot block drafters such as DFlash are especially attractive because they generate an entire draft block in one forward pass. Their limitation is that each future position is predicted from the prefix alone: the distribution for token `x_{t+i}` does not depend on the earlier drafted tokens. This marginal conditioning becomes increasingly mismatched with the verifier's true autoregressive distribution as draft depth grows, and it is especially limiting for tree drafting, where different branches need different continuations.

TreeFlash addresses this bottleneck with a small AR-approximation module. It conditions each draft position on the previous token while preserving the O(1) drafting time complexity that makes one-shot drafters fast.

## How It Works

<figure>
  <object class="paper-figure" data="/assets/treeflash/presentation_ar_approx.pdf" type="application/pdf">
    <a href="/assets/treeflash/presentation_ar_approx.pdf">View Figure 2</a>
  </object>
  <figcaption>
    <strong>Figure 2:</strong> TreeFlash modifies each one-shot hidden state with a lightweight SwiGLU layer conditioned on the previous token embedding. The adjusted hidden state is then projected with the verifier's output embedding to produce an AR-approximated token distribution.
  </figcaption>
</figure>

TreeFlash has three main components:

1. **One-shot backbone:** TreeFlash starts from a DFlash-style block drafter that predicts all draft positions in parallel.
2. **AR-approximation:** For each draft position, a small SwiGLU module combines the drafter hidden state with the embedding of the preceding token. This produces a corrected distribution `q'(x_{t+i} | x_{\leq t}, x_{t+i-1})`.
3. **Tree construction:** The corrected distributions are used with OPT-Tree selection to allocate the draft budget to the most promising candidate continuations.

Naively applying AR conditioning would require sequential steps. TreeFlash avoids this with a fixed top-M intermediate tree: because the AR-approximator conditions only on the immediately preceding token, all modified distributions can be computed in parallel across draft depth and candidate tokens.

Training initializes from a pretrained DFlash checkpoint and zero-initializes the AR-approximator, so training begins as a faithful copy of DFlash. The model is then fine-tuned with forward KL divergence against the verifier distribution, using random anchor positions and ground-truth previous tokens for the AR-approximation input.

## Results

<figure>
  <object class="paper-figure" data="/assets/treeflash/q3_speedup_graph.pdf" type="application/pdf">
    <a href="/assets/treeflash/q3_speedup_graph.pdf">View Figure 4</a>
  </object>
  <figcaption>
    <strong>Figure 4:</strong> Average speedup across datasets for standard sampling and greedy decoding. TreeFlash consistently improves over both DFlash and DDTree, and the gain increases when the draft budget grows from B=16 to B=64.
  </figcaption>
</figure>

TreeFlash is evaluated on Qwen3 4B, Qwen3 8B, and Qwen3 Coder 30B A3B across mathematical reasoning, code generation, and instruction-following tasks. Baselines include EAGLE-3, DFlash, and DDTree.

### Key numbers

- **+24.8% block efficiency** and **+17.1% speedup** over DFlash at the budget-matched setting B=16.
- **+7.5% block efficiency** and **+3.9% speedup** over DDTree at B=16, showing that AR-approximation adds value beyond tree construction alone.
- **+12.4% block efficiency** and **+9.1% speedup** over DDTree at B=64, where deeper tree positions make AR conditioning more important.
- **+46.2% speedup** over DFlash at B=64 on average across the evaluated target models.
- On Qwen3 Coder 30B A3B, TreeFlash improves over DFlash by **+1.50 tau** at B=16 and achieves **+42.8% speedup** over DFlash at B=64.

## Why AR-Approximation Helps

<figure>
  <object class="paper-figure" data="/assets/treeflash/tvd_position_graph.pdf" type="application/pdf">
    <a href="/assets/treeflash/tvd_position_graph.pdf">View Figure 5</a>
  </object>
  <figcaption>
    <strong>Figure 5:</strong> DFlash's total variation distance to the verifier grows sharply with draft depth. TreeFlash grows much more slowly, reaching 0.62 at depth 15 compared with 0.81 for DFlash.
  </figcaption>
</figure>

The core issue is not only that DFlash is imperfect; it is that marginal future-token distributions are structurally limited. Even the ground-truth marginal distribution diverges from the verifier's autoregressive distribution as draft depth increases. TreeFlash reduces this mismatch by conditioning on the previous token, capturing local token-level coherence that pure marginal distributions miss.

<figure>
  <object class="paper-figure" data="/assets/treeflash/coverage_position_graph.pdf" type="application/pdf">
    <a href="/assets/treeflash/coverage_position_graph.pdf">View Figure 6</a>
  </object>
  <figcaption>
    <strong>Figure 6:</strong> TreeFlash uses draft budget more efficiently at deeper positions. By depth 15, TreeFlash top-1 coverage slightly exceeds DFlash top-5 coverage.
  </figcaption>
</figure>

This improved calibration matters most in tree drafting. A tree budget is only useful when the selected branches remain plausible under the verifier. TreeFlash assigns probability mass to branch-specific continuations, so the draft tree can spend its budget on candidates that are more likely to survive verification.

## Qualitative Comparison

<div class="figure-grid">
  <figure>
    <object class="paper-figure" data="/assets/treeflash/qual_ddtree_partial.pdf" type="application/pdf">
      <a href="/assets/treeflash/qual_ddtree_partial.pdf">View DDTree example</a>
    </object>
    <figcaption><strong>DDTree:</strong> The marginal distribution induces nested branch structure because later token rankings are shared across paths.</figcaption>
  </figure>
  <figure>
    <object class="paper-figure" data="/assets/treeflash/qual_treeflash_partial.pdf" type="application/pdf">
      <a href="/assets/treeflash/qual_treeflash_partial.pdf">View TreeFlash example</a>
    </object>
    <figcaption><strong>TreeFlash:</strong> AR-approximation creates branch-specific continuations, producing a more coherent and better-utilized draft tree.</figcaption>
  </figure>
</div>

## Ablations

<figure>
  <object class="paper-figure" data="/assets/treeflash/abl_treesize_graph.pdf" type="application/pdf">
    <a href="/assets/treeflash/abl_treesize_graph.pdf">View Figure 7</a>
  </object>
  <figcaption>
    <strong>Figure 7:</strong> Increasing the draft budget improves block efficiency and throughput in the single-batch setting, with TreeFlash continuing to benefit from larger trees.
  </figcaption>
</figure>

<figure>
  <object class="paper-figure" data="/assets/treeflash/top_m_latency_acceptance_graph.pdf" type="application/pdf">
    <a href="/assets/treeflash/top_m_latency_acceptance_graph.pdf">View Figure 8</a>
  </object>
  <figcaption>
    <strong>Figure 8:</strong> The intermediate top-M tree trades off AR-approximation coverage against runtime overhead. In these experiments, throughput peaks M <= 32.
  </figcaption>
</figure>

<div class="callout">
  <strong>Key takeaway:</strong> TreeFlash improves one-shot speculative decoding by adding a lightweight conditioning mechanism, closing the gap between marginal and fully autoregressive distributions and resulting in higher acceptance and block efficiency.
</div>

## Code

The implementation is available on GitHub: [ETH-DISCO/TreeFlash](https://github.com/ETH-DISCO/TreeFlash).
