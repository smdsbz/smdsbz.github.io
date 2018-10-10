---
layout: article
title: Self-Normalizing Neural Networks Note
key: self-normalizing-neural-networks-note
tags: MachineLearning DeepLearning
---

Notes on [Self-Normalizing Neural Networks](https://arxiv.org/pdf/1706.02515.pdf).  

<!-- more -->

**Intuition**  

Only two design choices are available for the function $g$: (1) the activation function and (2) the initialization of the weights.  

Requirements for an SNN activation function
1.  negative and positive values for controlling the mean
2.  saturation regions (derivatives approaching zero) to dampen the variance if it is too large in the lower layer
3.  a slope larger than one to increase the variance if it is too small in the lower layer
4.  a continuous curve

Introducing the "Scaled Exponential Linear Units", or SELUs  

$$
\mathrm{selu}(x) = \lambda \begin{cases}
    &x                      &\text{if } x \gt 0     \\
    &\alpha e^{x} - \alpha  &\text{if } x \leq 0
\end{cases} \,.
$$

Now find the magic numbers $\lambda$ and $\alpha$ that make auto-convergence happen.  

---

*To Be Continued*
