---
layout: post
title: "Scaling Laws for LLM-Based Compression"
excerpt: "Investigating how large language models compress text, image, and speech with universal power laws"
date:   2025-07-28 10:00:00
mathjax: false
comments: false
---

### Introduction

The classic paper on [scaling laws for neural language models](https://arxiv.org/abs/2001.08361) showed that a model’s cross-entropy loss can be predicted from a power-law function of three quantities:
1. The dataset size
2. The number of model parameters
3. The training steps or compute budget

But if a model is very good at predicting the next token, it should also be very good at **compressing** the input. In fact, prediction and compression are mathematically equivalent via arithmetic coding. This is the core idea behind the Hutter Prize and AIXI — that intelligence is compression.

So I set out to ask a simple question:

> **Can we find clean scaling laws for data compression using large language models, across multiple modalities?**

---

### Experiment Setup

I used the [Pythia suite](https://arxiv.org/abs/2304.01373) of open-source language models, which offer multiple checkpoints across sizes (from 70M to 1.4B parameters), all trained on the same dataset (The Pile) — ideal for controlled comparisons.

To measure compression:
- I took the first 2048 chunks (each of 2048 bytes) from the `enwik8` dataset.
- For each chunk, I fed it into a Pythia model and used the predicted next-token probabilities as a **symbol table** for an arithmetic encoder.
- Since arithmetic encoding is near-optimal given predicted probabilities, the resulting byte size reflects how well the LLM "understands" the data.

---

### Results: Text Compression

I fit a Kaplan-style scaling law of the form:

\[
\text{CR}(P, S) = a + b \cdot P^{-\alpha} + c \cdot S^{-\beta}
\]

where:
- **P** = number of parameters
- **S** = training step
- **CR** = compression ratio
- **α**, **β** are fitted exponents

From the paper:

- **Text scaling fit coefficients**:
  - \( a = 1.00 \)
  - \( b = 0.44 \), \( \alpha = 0.38 \)
  - \( c = 0.14 \), \( \beta = 0.75 \)

![Text Compression Scaling](https://raw.githubusercontent.com/rokosbasilisk/prh-experiments/main/assets/text_scaling.png)
*Figure 1: Compression ratio vs model size and training step for text (enwik8).*

---

### Compression Beyond Text: Images and Speech

Inspired by the [“Language Modeling is Compression”](https://arxiv.org/abs/2306.03061) paper, I extended this experiment to **raw image and speech bytes**, without any modality-specific preprocessing.

The catch? These models were **never trained on image or audio data** — only text.

So I encoded RGB images and PCM waveforms into byte streams (e.g., ASCII) and passed them through the same compression pipeline.

#### Image Compression

- **Image scaling fit coefficients**:
  - \( a = 1.00 \)
  - \( b = 0.20 \), \( \alpha = 0.15 \)
  - \( c = 0.20 \), \( \beta = 0.61 \)

![Image Compression Scaling](https://raw.githubusercontent.com/rokosbasilisk/prh-experiments/main/assets/image_scaling.png)
*Figure 2: Compression ratio vs model size and training step for image byte streams (ImageNet).*

#### Speech Compression

- **Speech scaling fit coefficients**:
  - \( a = 1.00 \)
  - \( b = 0.17 \), \( \alpha = 0.17 \)
  - \( c = 0.11 \), \( \beta = 0.45 \)

![Speech Compression Scaling](https://raw.githubusercontent.com/rokosbasilisk/prh-experiments/main/assets/speech_scaling.png)
*Figure 3: Compression ratio vs model size and training step for speech byte streams (LibriSpeech).*

---

### Combined Scaling Laws

Below is a unified plot showing the scaling laws for text, image, and speech compression using LLMs:

![Combined Compression Scaling](https://raw.githubusercontent.com/rokosbasilisk/prh-experiments/main/assets/combined_scaling.png)
*Figure 4: Comparison of scaling laws for text, image, and speech compression.*

We observe that:
- Text has the strongest scaling behavior (steepest \(\alpha\) and \(\beta\))
- Image and speech still show clear trends, even without modality-specific training
- All curves follow a power-law decay in

