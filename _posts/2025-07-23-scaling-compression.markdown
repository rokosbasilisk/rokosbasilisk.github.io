---
layout: post
title: "Scaling Laws for LLM-Based Compression"
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
  1. Raw bytes → ASCII via `chr(b % 128)`  
  2. Tokenize (max_length = 1024)  
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
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.22</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.18</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.17</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.17</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.17</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-160M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.22</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.16</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.15</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.15</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.15</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-410M</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.22</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.15</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.14</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.13</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.13</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-1 B</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.21</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.14</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.13</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.12</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.12</td>
    </tr>
    <tr>
      <td style="border:1px solid #444;padding:4px;">pythia-1.4 B</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.21</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.14</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.12</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.11</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.11</td>
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
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.60</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.50</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.49</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.50</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.51</td>
    </tr>
    <!-- additional rows identical to above -->
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
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.69</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.46</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.44</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.47</td>
      <td style="border:1px solid #444;padding:4px;text-align:right;">0.47</td>
    </tr>
    <!-- additional rows identical to above -->
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

<div style="text-align:center;">
  <img src="/assets/scaling_laws_for_compression.png"
       alt="Scaling curves for text, image, speech"
       style="width:100%; height:auto;"/>
</div>

---

## Conclusion

It has been argued that LLMs trained only on text can form proto-AGI world models [Thoughts on the Alignment Implications of Scaling Language Models](https://bmk.sh/2021/06/02/Thoughts-on-the-Alignment-Implications-of-Scaling-Language-Models/). These compression results reinforce that claim: despite text-only pretraining, LLMs compress images and speech following clear power laws.

We hypothesize two primary mechanisms:

1. **In-Context Learning (ICL)**  
   Within a given token window, self-attention adapts on-the-fly to file-specific patterns (repeating pixels, periodic audio cycles), shaving off most of the entropy.

2. **Universal Sequence Prior (USP)**  
   Pretraining internalizes bursty repetitions, Zipf’s law, heavy tails, and long-range autocorrelations—statistics shared by all byte streams. Even without context, compression ratio should be far below uniform-noise rates.

Future work should quantify each component’s contribution and trace how they emerge during pretraining.

---

## References

- Kaplan, J.; McCandlish, S.; Henighan, T.; Brown, T.; Chess, B.; Child, R.; Gray, S.; Radford, A.; Wu, J.; Amodei, D. “Scaling Laws for Neural Language Models.” *arXiv* 2001.08361 (2020). [https://arxiv.org/abs/2001.08361](https://arxiv.org/abs/2001.08361)  
- The Intricate Link Between Compression and Prediction. Mindful Modeler Substack (2024). [https://mindfulmodeler.substack.com/p/the-intricate-link-between-compression](https://mindfulmodeler.substack.com/p/the-intricate-link-between-compression)  
- Delétang, G.; Ruoss, A.; Duquenne, P.-A.; Catt, E.; Genewein, T.; Mattern, C.; Grau-Moya, J.; Wenliang, L.K.; Aitchison, M.; Orseau, L.; Hutter, M.; Veness, J. “Language Modeling Is Compression.” *arXiv* 2309.10668 (2023). [https://arxiv.org/abs/2309.10668](https://arxiv.org/abs/2309.10668)  
- Gao, Leo. “Thoughts on the Alignment Implications of Scaling Language Models.” BMK.sh Blog (2021). [https://bmk.sh/2021/06/02/Thoughts-on-the-Alignment-Implications-of-Scaling-Language-Models/](https://bmk.sh/2021/06/02/Thoughts-on-the-Alignment-Implications-of-Scaling-Language-Models/)  
- Tao, T. “Benford’s Law, Zipf’s Law and the Pareto Distribution.” Terence Tao’s Blog (2009). [https://terrytao.wordpress.com/2009/07/03/benfords-law-zipfs-law-and-the-pareto-distribution/](https://terrytao.wordpress.com/2009/07/03/benfords-law-zipfs-law-and-the-pareto-distribution/)  

