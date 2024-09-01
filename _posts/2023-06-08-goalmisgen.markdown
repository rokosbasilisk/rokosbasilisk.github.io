---
layout: post
comments: false
title: "Goal-misgeneralization might be ELK-hard"
excerpt: "can goal-misgeneralization be formulated as an instance of ELK?"
date:   2023-03-24 10:00:00
mathjax: false
---

Goal misgeneralization and the ELK (Eliciting Latent Knowledge) problem are two of the most intriguing yet well-defined sub-problems within the broader AI alignment challenge. In this post, I'll share my opinion on why solving goal misgeneralization is as challenging as solving the ELK problem.

### Goal Misgeneralization

Goal misgeneralization occurs when a reinforcement learning agent retains its capabilities out-of-distribution yet pursues the wrong goal. This misalignment between the intended goal and the agent's actual behavior can lead to unexpected and potentially harmful outcomes.
A more detailed explanation of this can be found in the [here](https://arxiv.org/abs/2105.14111).

### The ELK Problem

Alignment Research Center (ARC) provides a simple, informal explanation of the ELK problem:

> Suppose we train a model to predict what the future will look like according to cameras and other sensors. We then use planning algorithms to find a sequence of actions that lead to predicted futures that look good to us.
> 
> But some action sequences could tamper with the cameras so they show happy humans regardless of what's really happening. More generally, some futures look great on camera but are actually catastrophically bad.
> 
> In these cases, the prediction model "knows" facts (like "the camera was tampered with") that are not visible on camera but would change our evaluation of the predicted future if we learned them. How can we train this model to report its latent knowledge of off-screen events?

A more sophisticated explanation of the ELK problem can be found in the [ELK problem document](https://docs.google.com/document/d/1WwsnJQstPq91_Yh-Ch2XRL8H_EpsnjrC1dwZXR37PC8/).

As of now, this problem too remains unsolved.

### Why Solving Goal Misgeneralization is as Hard as Solving ELK

Consider an adversarial training scheme for solving goal misgeneralization, such as Redwood Research's work on ["Adversarial Training for High-Stakes Reliability"](https://arxiv.org/pdf/2205.01663.pdf).

Let's examine a model trained to perform a specific task. To guarantee worst-case performance for this model, we need to establish bounds for its outputs in adversarial examples. However, running the model in the actual environment is not feasible due to potential unwanted consequences.

To address this, we can use a prediction-head to generate a simulated version of the trajectory that the model might undergo. We can then train a classifier from the output of the prediction-head to determine whether the AI system exhibits goal misgeneralization in an adversarial manner. The model can be penalized for misgeneralization behavior, and the resulting policy can be iteratively distilled into a newer model.

This prediction-head can be used to build another model that acts as a classifier, allowing us to distill the conservative policy (model) into a newer one.

However, this approach is only viable when the prediction-head is not deceiving, which is essentially the same challenge posed by the ELK problem.

In conclusion, the difficulty of solving goal misgeneralization for the worst case mirrors the core challenge of the ELK problem.
