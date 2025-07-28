---
layout: post
title: "Scaling Laws for LLM-Based Data Compression"
excerpt: "Investigating how large language models compress text, image, and speech with universal power laws"
date:   2025-07-23 10:00:00
mathjax: true
comments: false
---

## Introduction

The [scaling laws for neural language models](https://arxiv.org/abs/2001.08361) showed that cross-entropy loss follows a power law in three factors:

1. **Dataset size**  
2. **Model parameters**  
3. **Training compute** (steps or epochs)  

The lower the cross-entropy loss, the better the model’s next-token prediction. Since prediction and compression are deeply linked ([The Intricate Link Between Compression and Prediction](https://mindfulmodeler.substack.com/p/the-intricate-link-between-compression)), a lower loss should translate into better lossless compression. This raises the question:

> **Do LLMs exhibit clean power-law scaling in compression performance?**

---

## Experiment

For the experiment, I considered the Pythia models, which are available at multiple checkpoints and parameter sizes. All Pythia models are trained on the same dataset (The Pile), ensuring a uniform comparison. Following Delétang et al. (2023), The LLM’s predicted probabilities for each token are used to update the symbol table of an arithmetic encoder (a near-optimal entropy coder). I took the first 2048 chunks of the enwik8 dataset each chunk is 2048 bytes and fed them to the LLM to predict the next-token probabilities.

The other experimental parameters are:

- **Models:** EleutherAI Pythia family (8-bit quantized): 70 M, 160 M, 410 M, 1 B, 1.4 B parameters  
- **Checkpoints:** 1 k, 8 k, 32 k, 128 k, 143 k training steps  
- **Chunking:**  
  - **CHUNK_SIZE:** 2048 bytes  
  - **NUM_CHUNKS:** 2048 per dataset  
- **Compression pipeline:**  
  1. Raw bytes → ASCII 
  2. Tokenize and pass the ASCII symbols to LLM
  3. Arithmetic coding on LLM probabilities
  4. **CR** = compressed_bits / original_bits  
- **Datasets & Preprocessing:**  
  - **Text:** enwik8 (Wikipedia XML)  
  - **Image:** ImageNet-1k validation → 32 × 64 grayscale patches (uint8 bytes)  
  - **Speech:** LibriSpeech “clean” train.100 → 16 kHz PCM → int16 bytes  

---

## Results: Text Compression

<div style="overflow-x:auto;width:100%;">
<table style="width:100%;border:1px solid #444;border-collapse:collapse;">
  <thead>
    <tr>
      <th style="border:1px solid #444;padding:4px;text-align:left;">Model</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">1 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">8 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">32 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">128 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">143 k</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-70M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.223</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.176</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.17</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.173</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.175</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-160M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.218</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.159</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.149</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.149</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.150</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-410M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.223</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.148</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.136</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.129</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.128</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-1B</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.207</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.140</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.128</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.120</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.120</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-1.4B</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.207</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.137</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.124</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.115</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.115</td>
    </tr>
  </tbody>
</table>
</div>

### Text scaling-law fit

We fit a Kaplan-style power law  
$$
\mathrm{CR}(P,S)
= a + b\,P^{-\alpha} + c\,S^{-\beta}.
$$  
For **text**, the best-fit coefficients are:  
$$
\mathrm{CR}_{\text{text}}
= 0.10 + 60\,P^{-0.38} + 10\,S^{-0.75}.
$$

---

## Compression on Non-Text Modalities

The “Language Modeling Is Compression” paper shows that an LLM trained only on text can compress arbitrary byte streams [Language Modeling Is Compression](https://arxiv.org/abs/2309.10668). I applied the same pipeline to ImageNet-1k patches and LibriSpeech audio.

### Image Results

<div style="overflow-x:auto;width:100%;">
<table style="width:100%;border:1px solid #444;border-collapse:collapse;">
  <thead>
    <tr>
      <th style="border:1px solid #444;padding:4px;text-align:left;">Model</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">1 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">8 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">32 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">128 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">143 k</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-70M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.601</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.499</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.492</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.505</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.513</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-160M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.615</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.483</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.471</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.482</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.492</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-410M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.668</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.506</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.461</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.444</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.447</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-1B</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.601</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.470</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.456</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.436</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.440</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-1.4B</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.643</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.482</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.470</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.434</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.436</td>
    </tr>
  </tbody>
</table>
</div>

### Image scaling-law fit

$$
\mathrm{CR}_{\text{image}}
= 0.38 + 2\,P^{-0.15} + 60\,S^{-0.86}.
$$

### Speech Results

<div style="overflow-x:auto;width:100%;">
<table style="width:100%;border:1px solid #444;border-collapse:collapse;">
  <thead>
    <tr>
      <th style="border:1px solid #444;padding:4px;text-align:left;">Model</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">1 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">8 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">32 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">128 k</th>
      <th style="border:1px solid #444;padding:4px;text-align:right;">143 k</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-70M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.695</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.460</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.439</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.475</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.466</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-160M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.678</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.440</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.430</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.433</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.456</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-410M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.770</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.505</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.404</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.383</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.391</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-1B</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.677</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.424</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.444</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.376</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.384</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-1.4B</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.752</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.469</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.443</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.378</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.385</td>
    </tr>
  </tbody>
</table>
</div>

### Speech scaling-law fit

$$
\mathrm{CR}_{\text{speech}}
= 0.39 + 8\times10^{3}\,P^{-0.68} + 1\times10^{2}\,S^{-0.85}.
$$

---

## Combined Scaling Curves
The below plot shows the overall scaling laws plots across all the three modalities, While the scaling law trend is present in non-textual modalities, the compression is not as strong in text.
<div style="text-align:center;">
  <img src="/assets/scaling_laws_for_compression.png"
       alt="Scaling curves for text, image, speech"
       style="width:100%; height:auto;"/>
</div>

---

## Conclusion

It has been argued that LLMs trained only on text can form [world models for proto-AGI](https://bmk.sh/2021/06/02/Thoughts-on-the-Alignment-Implications-of-Scaling-Language-Models/). These compression results reinforce that claim: despite text-only pretraining, LLMs compress images and speech following clear power laws.

We hypothesize two primary mechanisms responsible for this:

1. **In-Context Learning**  
   Within a given token window, self-attention adapts on-the-fly to file-specific patterns (repeating pixels, periodic audio cycles), shaving off most of the entropy.

2. **Universal Sequence Prior**  
   Pretraining internalizes bursty repetitions, Zipf’s law, heavy tails, and long-range autocorrelations statistics shared by all byte streams. Even without context, compression ratio should be far below uniform-noise rates. Do read [Benford’s Law, Zipf’s Law and the Pareto Distribution](https://terrytao.wordpress.com/2009/07/03/benfords-law-zipfs-law-and-the-pareto-distribution/) to know more about naturally occuring data distributions and their universality

A really interesting future work should be to quantify each component’s contribution and trace how they emerge during pretraining.

---

## References

- Kaplan, J.; McCandlish, S.; Henighan, T.; Brown, T.; Chess, B.; Child, R.; Gray, S.; Radford, A.; Wu, J.; Amodei, D. “Scaling Laws for Neural Language Models.” *arXiv* 2001.08361 (2020). [https://arxiv.org/abs/2001.08361](https://arxiv.org/abs/2001.08361)  
- The Intricate Link Between Compression and Prediction. Mindful Modeler Substack (2024). [https://mindfulmodeler.substack.com/p/the-intricate-link-between-compression](https://mindfulmodeler.substack.com/p/the-intricate-link-between-compression)  
- Delétang, G.; Ruoss, A.; Duquenne, P.-A.; Catt, E.; Genewein, T.; Mattern, C.; Grau-Moya, J.; Wenliang, L.K.; Aitchison, M.; Orseau, L.; Hutter, M.; Veness, J. “Language Modeling Is Compression.” *arXiv* 2309.10668 (2023). [https://arxiv.org/abs/2309.10668](https://arxiv.org/abs/2309.10668)  
- Gao, Leo. “Thoughts on the Alignment Implications of Scaling Language Models.” BMK.sh Blog (2021). [https://bmk.sh/2021/06/02/Thoughts-on-the-Alignment-Implications-of-Scaling-Language-Models/](https://bmk.sh/2021/06/02/Thoughts-on-the-Alignment-Implications-of-Scaling-Language-Models/)  
- Tao, T. “Benford’s Law, Zipf’s Law and the Pareto Distribution.” Terence Tao’s Blog (2009). [https://terrytao.wordpress.com/2009/07/03/benfords-law-zipfs-law-and-the-pareto-distribution/](https://terrytao.wordpress.com/2009/07/03/benfords-law-zipfs-law-and-the-pareto-distribution/)  

