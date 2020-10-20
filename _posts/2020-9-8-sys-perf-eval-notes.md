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

* average number of customers in system $$q = E[N] = \rho / (1-\rho)$$
* $$Var[N] = \rho / (1-\rho)^2$$

> 结合切比雪夫不等式，对系统前缓冲器大小设置中有指导意义。

combined with Little's formula

* average response time $$T_q = q / \lambda = 1 / [\mu(1-\rho)]$$
* average number of customers waiting $$w = q - \rho = \rho^2/(1-\rho)$$
* average waiting time $$T_w = \rho / [\mu(1-\rho)]$$

__M/M/n queueing__

此时系统出流量最多为 $$n\mu$$

$$
\eta_{k-1}\lambda + \eta_{k+1}\max(n,k+1)\mu = \eta_k (\lambda + \max(n,k-1)\mu)
$$

$$
\begin{aligned}
\eta_k &= \begin{cases}
\eta_0 \rho^k \frac{1}{k!}, &k \lt n\\
\eta_0 \rho^k \frac{1}{n! n^{k-n}}, &k \geq n
\end{cases}\\
\eta_0 &= [\sum_{k=0}^{n-1}\frac{(n\rho)^k}{k!} + \frac{(n\rho)^k}{n!} \frac{1}{1-\rho}]^{-1}\\
q &= n\rho + \rho \frac{(n\rho)^n}{n!} \frac{\eta_0}{(1-\rho)^2}
\end{aligned}
$$

* __shared queue__ or independent queue?
    * eg. 2x M/M/1 vs M/M/2, M/M/2 win (by $$T_q$$)
        * in 2x M/M/1, one process with empty queue cannot serve customer waiting in the other queue
* shared queue + independent queue?
    * selection, followed by independent queue (supermarket model)
    * choosing shortest queue in $$d$$ samples vs random choose (2001, Dr. M. Mitzenmacher, Harvard University)
* single proessor multiple processes queueing system
    * M/M/m model

Queueing networks
----------------

### Jackson's network

* open queueing network
* M nodes (queueing system), independent, apply to exponential distribution of rate $$\mu_i(n)$$
* arrival from outside to node $$\gamma_i$$
* customer choose to enter other node or leave system with prob $$q_{ij}\ (i=\{1,\dots,M\},j=\{0,1,\dots,M\})$$

__Jackson's traffic equation__
* average arrival rate of note i $$\lambda_i = \gamma_i + \sum \lambda_j q_{ji}$$
* total traffic $$\gamma = \sum \lambda_i q_{i0}$$
    * $$q_{i0}$$ is prob of customer departure after getting serviced at node i

__Single data transfering channel__
* communication task stream $$\gamma$$
* transfer fail prob $$q$$

__Jackson's theorem (stable states distribution)__
* number of customers at node i is independent with that at other nodes
* total arrival at node i is a Poisson stream of rate $$\lambda_i$$, and

    $$
    \begin{aligned}
    &\eta(n_1, \dots, n_M) = \prod \eta_i(n_i) \\
    &\eta_i(n_1) = \eta_i(0) \frac{\lambda_i^{n_i}}{\prod_{j=1}^{n_i} \mu_i(j)}
    \end{aligned}
    $$

__Jackson's network measurements__
* mean number of customers at node i $$q_i = \rho_i / (1-\rho_i)$$
* mean response time (using Little's formula on equivalent single queueing system) $$T_q = \sum \rho_i / (1-\rho_i) / \gamma$$
* mean waiting time $$T_{wi} = (q_i - \rho_i) / \lambda_i$$
* interval from arrival at node i to departure $$T_{i0} = T_{qi} + \sum q_{ij}T_{j0}$$

> another viewing angle: mean visiting times at node i $$v_i$$
>
> * $$v_i = \lambda_i / \gamma = \gamma_i / \gamma + \sum_{j=1}^M v_j q_{ji}$$
> * $$q_{0i} = \gamma_i / \gamma$$
> * $$T_{qi} = \frac{D_i}{v_i(1-\gamma D_i)}, \quad D_i = \rho_i / \gamma$$

### M/G/1

泊拉前克-欣钦公式 Pollaczek-Khintchine

$$
T_w = \frac{\lambda}{2 (1-\lambda \underbrace{T_S}_\text{平均单次处理时间})} E[{\underbrace{T_s}_\text{一次正确服务时间}}^2]
$$

### Sequential network

_可看作 Jackson 网络特例_

每个节点独立，i.e. 系统状态为联合分布，具有乘积解

### Gordon-Newell network

* like Jackson's network, but with $$q_{0j} = q_{i0} = 0$$
* closed system, constant customers $$K$$
* state space: $$S = \{ (n_1, \dots, n_M) | n_i \geq 0, \sum n_i = K \},\ |S| \lt \infty$$

<br/>

* traffic equation $$\lambda_i = \sum \lambda_j q_{ji}$$
* for one non-zero solution $$(e_i,\dots,e_M)$$, $$\lambda_i = c \cdot e_i$$
* stable state distribution

    $$
    \begin{aligned}
    \eta(n_1,\dots,n_M) &= \frac{1}{G} \prod_{i=1}^M x_i(n_i) \\
    x_i(n_i) &= \frac{e_i^{n_i}}{\prod_{j=1}^{n_i} \mu_i(j)},\ G(M,K): \text{unitary constant}
    \end{aligned}
    $$

* closed network equivalent open network
    * insert node $$o$$, where $$\lambda_{0o}=\mu_{o0}$$

### Approximative solution of queueing networks

* limited queue length
* priority scheduling
* non-exponential service distribution

lead to local unbalance, 解不具有乘积解形式

__Closed / open network with limited resources__

* upper bound modification $$R_U$$ 悲观修改 吞吐量最小
    * 阻塞加挡/吞吐率加墙：when output is 0, let input be 0 too
    * when node 2 is full, let node 1's input be blocked too
    * when node 1 is full, let node 2's departure be blocked simultaneously
* lower bound modification $$R_L$$ 乐观修改 吞吐量最大
    * to avoid output being 0, don't let arrival be blocked 
    * only when the whole system is full, refuse new customers

$$
\begin{cases}
&R_U: n_1 \leq N_1, n_2 \leq N_2, n_1 + n_2 \neq N_1 + N_2 \\
&R_O: n_1 \leq N_1, n_2 \leq N_2 \\
&R_L: n_1 + n_2 \leq N_1 + N_2
\end{cases}
$$

__Shared resource system__

* upper bound modification
    * while server 1's input is blocked, let output be blocked too
* lower bound modification
    * cacel server 2's priority, shares D equally

__Decomposing method__

Markov's chain (MC) is NCD (Nearly Complete Decomposable, 接近完全分解)

* iff MC's state transferring rate matrix is approx. to diagonal __blocks__ matrix
* (unitizing block) 置零位置的概率补充到子系统中，满足概率之和为 1
* (states compressing) condition: sub-sums of each row are same
    * get Concentrating matrix 集总矩阵

> May rearrange states' order, making NCD essentially __states groups'__ decomposing.

----------------------------------------------

Petri Nets
==========

* graphical description of information system

Model description
-----------------

* Place (circle)
* Transition (vertical line)
* Directed arc
* Token (dot in circle)

__Firing rule 实施规则__

* Enable condition: each input place has at least w token
* Fired results: input place decreases w token, output place increases w token
* Atomic operation: tokens moving out from input place and moving into output
    place cannot be separated

> __NOTE:__ 同一变迁下的弧权可能不同
>
> ```text
> O <-- w1 -- | <-- w2 -- O ,  w1 != w2
> ```

__Smallest token set 最小标识集 $$[M_0\gt$$__

系统可能达到的所有状态的集合，可达标识集 reachable token set

__Reachable tree__

TODO

__Associated matrix, invariant__

TODO