---
layout: article
title: Computer System Analysis and Performance Evaluation
key: sys-perf-eval-notes
tags: System PerformanceEvaluation CourseNotes
---

计算机系统分析与性能评价课程笔记

<!-- more -->

学科意义与方法论
=============

__Objects 目标__

* Performance abstraction, representation & conprehensive analysis

    e.g. 人 = 德 + 智 + 体 + ...

* Optimally designing systems / selection

* Queueing theory, Petri nets & conbination of tests and simulations


__方法选择__

* Measurements

    given system (post-prototype), given workload

* Simulation modeling

    system / environment not given

* Analytical modeling

    揭示系统内在规律，指明工作理论性贡献，但须结合实际实验数据！

    probability, stochastic process, queueing systems, queueing networks,
    Petri nets, Markov-chain, etc.

    因负载/任务随机性，多采用随机过程与概率方法。


__Why?__

* Meet intended application
* Meet efficiency & reliability requirements
* To design / build / operate systems near its optimal level of processing
    power under given requirements


__Measures 度量__

* Response time _(definition varies)_
* Waiting time, processing time

    queueing vs. serving

* Utilization
* Throughput
* Missionability 可用性

    If system remain operational for entire duration of a mission.

    * Reliability

        Probability of system performing correctly throughout a mission.

* Dependability

    Reliability over long run.

    Number of failures / day, MTTF, MTTR, long-term availability

> __Reliability vs. Dependability__
>
> Reliability
> * Useful for system where failure is catastrophic
> * Focusing on short-term mission.
>
> Dependability
> * Useful for systems where repair is possible and failure is tolerable.
> * Focusing on long-term overall bahavior.


__Technique__

* Measurements

    1. Design experiment.
    2. Gather data of performance parameters (hardware / software / hybrid).
    3. Statistical analysis to draw meaningful concolusion.

* Simulation modeling

    1. Construct system behavior model.
    2. Drive the model with appropriate abstraction of workload.
    
    Involving experiment design, data gathering & analysis.

* Analytic modeling

    1. Construct mathematical model

        Queueing, Markov-chain, Petri, etc.

    2. Solve It!

__A Systematic Approach__

1. State goals and define system
2. List system services and outcomes
3. Select performance metrics
4. List parameters that affect performance
5. Select factors to study
6. Select evaluation technique
7. Select workload
8. Design experiment
9. Analyze and interpret data

__条件概率__

* 条件概率

    $$
    P(B|A) = \frac{P(AB)}{P(A)}
    $$

* 事件独立

    $$
    P(AB) = P(A) P(B)
    $$

设 $$\Omega = E_1 + \dots + E_n$$，则

* 全概率公式

    $$
    P(A) = \sum_i{P(A|E_i)P(E_i)}
    $$

* 贝叶斯公式（后验概率公式）

    $$
    P(E_i|A) = \frac{P(A|E_i)P(E_i)}{\sum_j{P(A|E_j)P(E_j)}}
    $$

    e.g. 线路传输问题，由以往数据传输统计数据，计算收到结果的可靠性。

__离散概率分布__

* 伯努利分布（0-1分布、两点分布）
* 二项分布

    $$
    P\{X=k\} = C_n^k p_k (1-p)^{n-k}, \quad 0 \leq k \leq n
    $$

* 几何分布（伯努利试验首次成功的试验次数）

    $$
    P\{X=m\} = p (1-p)^{m-1}, \quad m = 1, 2, \dots
    $$

* __泊松分布__

    $$
    P\{X=k\} = \frac{\lambda^k}{k!} e^{-\lambda}, \quad k=0,1,\dots
    $$

    期望与方差均为 $$\lambda$$。

* 正态分布
* Γ分布

泊松分布与泊松过程
--------------

__泊松事件流（泊松过程）__

* 平稳性

    事件发生概率与考虑的微小时间段 $$\Delta t$$ 成正比，但与起起始与终止位置无关

* 稀有性

    在微小时间段发生 2 次以上事件的概率 → 0

* 无后效性

    不同时间段的事件相互独立

* 微分性

    $$P\{X(t)=n\}$$ 关于 $$t$$ 可微

__泊松事件流与泊松分布的关系__

对于泊松事件流，在长为 $$t$$ 的时间间隔内事件出现的次数 $$v(t)$$ 服从参数为 $$\lambda t$$ 的泊松分布。

__泊松过程与指数分布关系__

泊松过程中观察任务到达时间间隔 $$T_a$$，则

$$
\begin{aligned}
P\{T_a < t\} &= 1 - P\{T_a \geq t\} = 1 - e^{-\lambda t } &\text{（t内多个任务到达）}\\
E(T_a) &= \frac{1}{\lambda}
\end{aligned}
$$


随机过程
------

__随机过程__

$$\{X(t), t \in \text{状态集（参数集、索引集）}\ T\}$$

* 离散参数（$$t$$）的随机过程：随机序列
* 离散状态（$$X(t)$$）的随机过程：链

__马尔可夫过程__

无后效性的随机过程

$$
P\{X(t) \leq x | X(t_n) = x_n, \dots, X(t_1)=x_1\}
= P\{X(t) \leq x | X(t_n)=x_n\}
$$

> 泊松过程为马尔可夫过程的特例。

-----------------------------------------------------------------

Measurement
===========

__Technique__

* Types of workloads
    * Instruction Mixes
        * average instruction time
        * performance of processor __only a factor__ of entire system performance
    * Processing Kernels
        * generalization of Instruction Mixes
    * Application Benchmark
* Performance monitoring
    * to measure resource utilizations & to find performance bottleneck
    * Event, Trace, Overhead, Domain, Input rate, Input width, Resolution
    * Event-driven, Timer-driven (Sampling)
* Summarizing measured data
    * Sample Mean / Median, Geometric Mean, Variability
* Experimental design
    * Full Factorial Design 全因子设计

-----------------------------------------------------------------

Queueing Theory
===============

__Properties of Queues__

* Arrival Process (A) 输入到达过程
    * arrival rate $$\lambda$$
    * waiting time $$T_w$$
* Service Time Distribution (S) 服务时间
    * service time $$T_s$$, service rate $$\mu=T_s$$
    * utilization $$\rho$$
    * 即使处理能力确定，但服务时间可能在不同时刻不一样（考虑缓存情况），但认为其分布确定

> time spent in system $$T_q = T_w + T_s$$

* Number of Servers (m)
* System Capacity (B) 系统容量
* Population Size (K)
* Service Discipline (SD)

Inter-arrival and service times are typically
* M Exponential, memoryless (Markov's process)
* D Deterministic, times are constant
* G General, distribution not specified

Simple queueing system
---------------------

### Queue

__Single server__

* $$\rho = \lambda T_s$$
* $$q = w + \rho$$
* maximum arrival rate $$\lambda_\text{max}=\frac{1}{T_s}$$
* departure rate $$\min{\{\lambda, \lambda_\text{max}\}}$$

__Multiple servers (single / multiple queues)__

* $$\rho = \lambda / n \cdot T_s$$
* $$q = w + n \rho$$
* maximunm arrical rate $$\lambda_\text{max}=\frac{n}{T_s}$$
* departure rate $$\min{\{\lambda, \lambda_\text{max}\}}$$

__General formula (Little's formula)__

* 系统中平均用户数 $$q = \lambda T_q$$
    * 即从用户进入系统到出系统时间中，平均进入系统的新用户人数
* 队列中平均等待用户数 $$w = \lambda T_w$$

### M/M/n modeling

_arrival Markov 泊松分布, serve Markov 负指数分布, n servers_

__M/M/1 modeling__

Assume $$N(t)$$ be number of customers in system at time $$t$$,
$$\{N(t), t \geq 0\}$$ is continuous and homogeneous Markov's link (birth and
death process)

> 连续时间齐次马尔科夫链：状态转移仅与时段有关，与初始状态无关。
>
> $$p(X_s + T = j | X_s = i) = f(T)$$

* server utilization $$\rho = \lambda / \mu$$
* stable states distribution $$\{\eta_k, k \geq 0\}$$

    $$
    \begin{aligned}
    &\underbrace{\eta_{k-1} \lambda + \eta_{k+1} \mu}_\text{State k's generation rate} = \underbrace{\eta_k (\lambda + \mu)}_\text{State k's departure rate}\\
    \Rightarrow\ &\eta_k = \rho^k(1-\rho)
    \end{aligned}
    $$

* $$q = E[N] = \rho / (1-\rho)$$
* $$Var[N] = \rho / (1-\rho)^2$$

* $$T_q = q / \lambda = 1 / [\mu(1-\rho)]$$
* $$w = q - \rho = \rho^2/(1-\rho)$$

Queueing networks
----------------
