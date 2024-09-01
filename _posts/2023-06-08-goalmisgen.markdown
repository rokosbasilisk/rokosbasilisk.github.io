---
layout: post
comments: false
title: "Goal-misgeneralization might be ELK-hard"
excerpt: "can goal-misgeneralization be formulated as an instance of ELK?"
date:   2023-03-24 10:00:00
mathjax: false
---


Consider an adversarial training-scheme for solving goal-misgeneralization, ( here i consider Redwood Research's work on ["Adversarial Training for High-Stakes Reliability"](https://arxiv.org/pdf/2205.01663.pdf)).

Consider a model that was trained to perform a specific task. To guarantee worst-case performance for this model, we need to have bounds for its outputs in adversarial examples. However, it is not feasible to run the model in the actual environment because it could lead to unwanted consequences.

Therefore, we can use a prediction-head to generate a simulated version of the trajectory that the model might undergo. We can train a classifier from the output of the prediction-head to classify whether the AI system exhibits goal-misgeneralization in an adversarial manner. The model can then be penalized for misgeneralization behavior, and the policy obtained can be distilled into a newer model iteratively. We can use this prediction-head to build another model that can act as a classifier and distill the conservative policy (model) into a newer one.

However, this is only possible when the prediction-head is not deceiving, which is essentially the same as the [ELK problem](https://docs.google.com/document/d/1WwsnJQstPq91_Yh-Ch2XRL8H_EpsnjrC1dwZXR37PC8/).
