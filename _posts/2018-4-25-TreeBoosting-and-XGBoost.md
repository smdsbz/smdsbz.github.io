---
layout: article
title: Tree Boosting and XGBoost
tags: MachineLearning
---

Learning notes of Boosted Tree and XGBoost.  

{:toc}

-----------------------------------------

## Boosted Tree

**Material**
- 	[Introduction to Boosted Trees](http://homes.cs.washington.edu/~tqchen/pdf/BoostedTree.pdf), slide by Tianqi Chen  

### Additive Approach

#### Intuition

Having $$N​$$ vectors, *i.e.* training set, with $$D​$$ features $$\vec{x}_n = (x_n1, x_n2, \dots , x_{nD})​$$, learn multiple classifiers somehow similar to *decision tree* denoted as $$f_{t}(\vec{x})​$$. Your final model output will be

$$
\hat{\vec{y}}(\vec{x}) = \sum_{t=1}^{T} f_{t}(\vec{x}) \\
\begin{align*}
\text{where}\>
    T &= \text{total amount of trees}
\end{align*}
$$

#### How to Learn

Define prediction for training round $$t \in \left[1, T\right]$$ and for sample $$\vec{x_i}$$ as

$$
\begin{align*}
\hat{\vec{y}}_{i}^{(t)}(\vec{x}_i)
    &= \sum_{k=1}^{t-1} f_{k}(\vec{x}_i) + f_{t}(\vec{x}_i) \\
    &= \hat{\vec{y}}_{i}^{(t-1)}(\vec{x}_i) + f_{t}(\vec{x}_i) \\
\end{align*}
$$

or in other words...

$$
\text{new_forest} = \text{existing_forest} + \text{new_tree}
$$

Note you **only** optimize the weak classifier $$f_{t}(\vec{x})$$, *i.e.* $$\text{new_tree}$$, *w.r.t.* current training round.  

After choosing a suitable differentiable convex loss function (or error function) $$l(\vec{y}_i, \hat{\vec{y}}_i)$$ and regularization term $$\Omega(f_{t})$$, usually

$$
\Omega(f_{t}) = \gamma T + \frac{1}{2} \lambda \sum_{j=1}^{T}w_{j}^{2} \\
\begin{align*}
\text{where}\>
    \gamma  &= \text{number of leaves}, \\
    \lambda &= \text{regularization parameter}, \\
    w_j     &= \text{score on}\> j\text{-th leaf}
\end{align*}
$$

And you basically do the following to get the new tree

$$
\begin{align*}
    f_{t}(\vec{x})
        &= \mathrm{argmin}_{f_t}\> \Big[ l(\vec{y}_i,\> \hat{\vec{y}}_i) + \Omega(f_{t}) \Big] \\
        &= \mathrm{argmin}_{f_t}\> \Big[ l(\vec{y}_i,\> \hat{\vec{y}}_i^{(t-1)} + f_{t}(\vec{x}_i))
            + \Omega(f_{t}) \Big] \\
\end{align*}
$$

#### Greedy Learning of Trees

For there are infinite different tree structures, we would want to learn the structure progressively by adding *splits*.  

Let's first define the advantage, or $$\text{Gain}$$, of adding a split as

$$
\text{Gain}
    = \frac{G_L^2}{H_L+\lambda} + \frac{G_R^2}{H_R+\lambda}
        - \frac{(G_L+G_R)^2}{H_L+H_R+\lambda} - \gamma \\
\begin{align*}
\text{where}\>
    G, H    &= \text{first and second order Taylor expansion of gradient respectively,} \\
    \lambda &= \text{regularization parameter,} \\
    \gamma  &= \text{complexity of model introduced by adding this split}
\end{align*}
$$

Sadly though, the $$\text{Gain}$$ only tells you how to **judge** a tree structure, and the only approch to actualy **grow** the trees is by **traversing**, with a time complexity of $$O\Big(N D K \log(n)\Big)$$, where $$K$$ is the depth of the tree grown.  

----------------------------------------

## XGBoost

**Material**
- [XGBoost: A Scalable Tree Boosting System](https://arxiv.org/pdf/1603.02754.pdf), T. Chen, C. Guestrin et. al.

XGBoost, *abbr.* eXtreme Gradient Boosting, is therefore devoloped to save the days of those good-ol' tree boosting algorithms.  

### Hacks

#### Approximate Algorithm

Batch-split version of the original **Exact Greedy Algorithm**, which was discussed in [last section](#greedy-learning-of-trees).  

This method saves it by telling you that filling training set into memory **all at once** is not neccessary.  

#### Weighted Quantile Sketch

Instead of iterate over exactly all possible split, define a rank function which serves as a criterion of choosing candidate split. The rank function is defined as follow:

$$
r_k(z) = \frac{1}{\sum_{(x, h) \in D_k}} \sum_{(x, h) \in D_k, x \lt z} h
$$

Note this would require the dataset to be sorted.  

#### Sparsity-aware Split Finding

Due to certain reasons, the input training sample $$\vec{x}$$ could be sparse, that is having missing values in the training input matrix $$X$$.  

XGBoost handles this by learning an optimal default direction with max $$\text{Gain}$$, before starting actual iteration over dataset (could be batches).  

#### Column Block for Parallel Learning

**Advantages**

- Stored Out-of-core and distributed.
- Collecting statistics for each column can be *parallelizes*.

#### Cache-aware Access

Good choice of *block size* counts!

#### Blocks for Out-of-core Computation

Mainly two techniques:  

**Block Compression**  
Blocks are compressed and decompressed on the fly.  

**Block Sharding**  
Shard data onto multiple disks, sounds like *RAID* to me.  

--------------------------------------------------

## My *Possibly Biased and Inaccurate* Summary

After reading the XGBoost paper, it seems that XGBoost is nothing new, but a new approch of improving real-world machine learning challenges, which is by utilizing **every possible piece** of those ACM and traditional computer science tricks. It's an excellent example of how old things comming together to become something new and dope again!  
