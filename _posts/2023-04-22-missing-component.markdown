---
layout: post
comments: false
title: "Missing Component for AGI"
excerpt: "AGI stream of thought"
date:   2023-04-22 10:00:00
mathjax: false
---


Current efforts in developing AGI systems primarily focus on two key aspects. The first involves establishing a prior understanding of the world's states, enabling predictions of subsequent states based on the current action or input, often referred to as a next-token or next-bit predictor.

The second aspect revolves around implementing planning algorithms or policies that utilize this prior knowledge to proactively anticipate and navigate future states. However, it's important to note that these planning algorithms work effectively only when the world-state is precisely defined, which isn't always the case in scenarios with imperfect information, such as certain games.

Nonetheless, a crucial missing component for achieving AGI is the development of robust sensor systems that can effectively capture and describe observations about the real world, thereby informing the world model. While text-based models like large language models (LLMs) rely on predefined data spaces for this third component, there is a question about whether this approach can scale to reach superhuman capabilities.

To propel a recursively self-improving system closer to AGI, it must tackle the challenges of robotics. Relying solely on a discrete text-based next-token predictor through repeated code generation and self-play would be a highly inefficient way to solve robotics problems. However, a highly capable next-bit predictor has the potential to emulate both a vision-based perceptual system capable of solving robotics tasks and a text-in-text-out language modeling component.

Crucially, this advanced model must be able to distinguish between different types of states it encounters from sensor readings. 

This discrimination could be facilitated through the use of separation tokens or other learned mechanisms, allowing the model to represent and manipulate multiple embeddings within the same space and dimensionality. This capability, akin to the concept of "meta" as discussed in the paper [megabyte](https://arxiv.org/pdf/2305.07185.pdf), would be a fundamental step towards achieving AGI.
