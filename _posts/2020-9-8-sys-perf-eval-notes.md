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

* Queuing theory, Petri nets & conbination of tests and simulations


__方法选择__

* Measurements

    given system (post-prototype), given workload

* Simulation modeling

    system / environment not given

* Analytical modeling

    揭示系统内在规律，指明工作理论性贡献，但须结合实际实验数据！

    probability, stochastic process, queuing systems, queuing networks,
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

    queuing vs. serving

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

        Queuing, Markov-chain, Petri, etc.

    2. Solve It!

----------------------------------------------------------------------
