---
layout: article
title: Streaming Media Technology
key: streaming-media-notes
tags: StreamingMedia MachineLearning DeepLearning CourseNotes
---

流媒体技术课程笔记

<!-- more -->

__Two learning manners__

* Bottom-up

    stimulus-driven, mostly obtained from early features, task independent
    
* Top-down

    related to recognition processing, influenced by prior knowledge (e.g.
    tasks to be performed, feature distribution of target, context of visual
    scene, etc.)

------------------------------------------------

Saliency
========

* __Saliency Detection__

Evaluation

<small>[Classification: ROC Curve and AUC](https://developers.google.cn/machine-learning/crash-course/classification/roc-and-auc)</small>

* ROC

    $$
    \text{TODO:}\ Alg.
    $$

* AOC: area under curve
    * shuffled AUC (sAUC) [Zhang et al.]: eliminating _edge effect_

__Salient Object Detection__

Evaluation

<small>[Classification: Precision and Recall](https://developers.google.cn/machine-learning/crash-course/classification/precision-and-recall)</small>

<small>[F-Score Definition](https://deepai.org/machine-learning-glossary-and-terms/f-score#:~:text=The%20F-score%20is%20a%20set-based%20measure%2C%20meaning%20that,a%20good%20overview%20of%20the%20search%20engine%E2%80%99s%20performance.)</small>

* PR curve: precision, recall, and F-measure

------------------------------------------------

SLIC Superpixels Segmentation
============================

__Application__
* Manifold ranking 流形排序
* Absorbing Markov-chain

__Intuition__

受位置约束的颜色空间 K-means 分类

1. uniform seeding
2. calculate distance, and label after argmin seed

    $$
    D(k,i) = d_{lab}(k,i) + \frac{m}{S} d_{xy}(k,i)
    $$

3. enforce connectivity
    * remove small isolated regions

------------------------------------------------

Semi-Supervised Learning
=======================

* exploit prior knowledge from unlabeled data

__Local and Global Consistency (LGC)__

Assumption

* nearby samples are likely having same label
* samples on same structure (refered to as cluster / manifold) are likely having
    same label

Take assumption to math...

cost function / energy function (*proposed, may have other forms)

$$
\begin{aligned}
&E(F) = \underbrace{\sum_i(F_i - Y_i)^2}_\text{label fitness} + \underbrace{\frac{1}{2} \lambda \sum_{ij} w_{ij} (\frac{F_i}{\sqrt{d_{ii}}} - \frac{F_j}{\sqrt{d_{jj}}})^2}_\text{manifold smoothness} \\
\Rightarrow &F \sim (I - \frac{\lambda}{1+\lambda}D^{-\frac{1}{2}}WD^{-\frac{1}{2}})^{-1} Y
\end{aligned}
$$

BP Network
==========

* 若希望输出和原始输入一样，则为常见的自编码网络
    * 正交矩阵，输入特征正交性较好，互不相关
