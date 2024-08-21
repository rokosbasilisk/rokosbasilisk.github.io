---
layout: post
title: "Decoding hidden chain of thought"
excerpt: "chain-of-thought is decryptable"
date:   2024-08-11 01:49:00
mathjax: false
comments: false
---

The paper [*"Let's Think Dot by Dot"*](https://arxiv.org/abs/2404.15758) introduces an intriguing approach to Chain of Thought (CoT) reasoning in language models. The authors trained a small 34M LLaMA model on CoT sequences for 3SUM and 2SUM tasks, where the sequences were replaced with hidden characters ("."). Notably, the model performs well on the 3SUM task with the hidden CoT but fails without it.

While the explicit training on these hidden sequences may seem unnatural, it raises an important question: Can we decrypt and understand what's happening "under the hood" during the hidden character CoT process?

The significant performance drop observed without the hidden CoT suggests that meaningful computations are occurring beneath the surface. To investigate this phenomenon, I conducted some small experiments aimed at answering two key questions:

What information is contained in the logits during this chain-of-thought?
How does the hidden CoT information propagate through the model's layers?

Initial findings:

During greedy decoding, while the top-ranked token is always a hidden character, lower-ranked tokens (rank 2 and below) contain sequences without hidden information. This suggests that the model maintains access to the original, unhidden information throughout the process.

Using a logit-lens-like method, I observed that the initial layers still contain the pure number sequences related to the 3SUM's CoT. These are gradually replaced with hidden characters in subsequent layers.
These preliminary results indicate that the model retains and processes the original information, even while outputting hidden characters. Further experiments are needed to gain additional insights into this intriguing phenomenon.

![*Hidden COT decoded tokens*](/assets/hiddencot.png)

For almost all the test-set examples, the hidden-tokens in the output appear starting from the 3rd (h3_out) layer

A modified greedy autoregressive decoding method was implemented. When decoding a "hidden-token" ("."), the algorithm selects the second-highest probability token instead of the top candidate. Results show 100% match in 3SUM satisfaction between this method and standard non-skip decoding.
