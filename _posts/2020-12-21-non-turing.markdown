---
layout: post
comments: true
title:  "human brains are turing-complete"
date:   2020-12-21 10:00:00
mathjax: false
---



I recently got to read a whitepaper on [neuromorphic computing](/assets/docs/non_turing.pdf), The core-thesis of the paper that the current "Turing-machine" formulation could'nt explain the cognitive capabilities human-brains, due to which computers cannot exhibit human-level learning ability. The post is my opinion about why this may be false.

<div class="imgcap">
<img src="/assets/turing_machine.jpeg">
<div class="thecap">An Artistic illustration of a turing-machine</div>
</div>
**A quick intro to turing-machines:** "Turing Machine" is hypothetical model of a computer, it is fundamental for understanding what a computer can and cannot do. Although the computers we use today do not entirely behave in the way an ideal turing machine is supposed to ("*infinite memory, single input tape etc..*"). They are found to be pretty useful especially in formalizing computer-science. 

**What is neuromorphic signal processing?**  It is a model of computing which is non-algorithmic  in nature. I.e, There is no concept of clear symbolic-arithmetic which is what makes human-brains "superior" when compared to turing machines. According to the paper key reasons for taking a departure from turing-computability are given as follows:-

1. In neural dynamics there are no clear candidates for symbols in the first place. This is
   due to the unclocked, parallel, distributed, high-dimensional, non-stationary and
   stochastic nature of neural dynamics which makes it hard to identify phenomena
   which could be called symbols. While a number of neurodynamical mechanisms have
   been suggested as carriers for symbols, they are disconcertingly different from each
   other and each of them lacks some of the properties of platonic symbols (discrete
   identifiability, unchangeability, addressability by name, finite alphabet.

   ​

2. Virtually all neural "processing rules" that have been suggested are formulated in
   non-symbolic terms using tools from dynamical systems theory or stochastic
   processes.

**why Neuromorphic signal processing may not be necessary** These are some reasons i can think of why neuromorphic-signal-processing is not necessary for building machines that can learn like humans.

1. It is found out that all the electrical-activity in neurons are via ionic currents through neuronal-membranes. These transmembranic currents involve four ionic species (sodium (Na+), potassium (K+), calcium (Ca2+ ), or chloride (Cl−)) . The electrical signals are propagated by changing the concentrations of these ions on both sides of the membranes of the cell which creates "electrochemical gradient". It is clear that to create a current it requires varying of concentration of Anion/Cations. A complete description of this can be given by assigning symbols to each ion and formulating the transport equations in terms of these variables without loss of information.

2. Even if all the processes in the brain are purely stochastic in nature, all these can be **efficiently** simulated on a turing-machine. No evidence is found out that the class of problems that can be solved using randomness ([BPP](https://en.wikipedia.org/wiki/BPP_(complexity))) is more powerful than those which can be solved without using randomness ([P](https://en.wikipedia.org/wiki/Time_complexity#Polynomial_time)).

3. In order to empirically establish that biological-brains are superior than turing-machines, we have to find a class of functions that can be solved by brains which are undecidable, of which there is no evidence so far.
