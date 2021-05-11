---
layout: post
comments: true
title:  "On Human Brains and Turing-Completeness"
excerpt: "My views on why human-brains can be turing-complete"
date:   2021-01-02 10:00:00
mathjax: false
---

<!-- 
<svg width="800" height="200">
	<rect width="800" height="200" style="fill:rgb(98,51,20)" />
	<rect width="20" height="50" x="20" y="100" style="fill:rgb(189,106,53)" />
	<rect width="20" height="50" x="760" y="30" style="fill:rgb(77,175,75)" />
	<rect width="10" height="10" x="400" y="60" style="fill:rgb(225,229,224)" />
</svg>
 -->

This is the part where you introdue the topic  [learn to play ATARI games](http://www.nature.com/nature/journal/v518/n7540/abs/nature14236.html).

<div class="imgcap">
<img src="/assets/rl/preview.jpeg">
<div class="thecap">Example image<b>From left to right</b>:some long description</div>
</div>

This is how you give points

1. One
2. Two


**Example heading**. Describe things here and give the code below. 

The code looks something like this:-
```python
h = np.dot(W1, x) # compute hidden layer neuron activations
h[h<0] = 0 # ReLU nonlinearity: threshold at zero
logp = np.dot(W2, h) # compute log probability of going up
p = 1.0 / (1.0 + np.exp(-logp)) # sigmoid function (gives probability of going up)
```



*italic things: look like this.* somethings like note.

