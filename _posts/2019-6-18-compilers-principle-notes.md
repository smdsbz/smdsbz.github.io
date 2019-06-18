---
layout: article
title: Compiler Principle Notes
key: compiler-principle-notes
tags: Compilers CourseNotes
---

编译原理复习笔记。

<!-- more -->

[TOC]

# 文法和语言

- 文法类型

    - 0 型文法（短语文法）

        设文法 $G = (V_N, V_T, P, S)$，如果任意 $\alpha \rightarrow \beta \in P$，__$\alpha$ 中至少包含一个非终结符__，则称文法 $G$ 属于 0 型文法。

    - 1 型文法（上下文有关文法）

        设文法 $G = (V_N, V_T, P, S)$，如果任意 $\alpha \rightarrow \beta \in P$，$\alpha$ 中至少含有一个非终结符，__且除空规则外，$\alpha$ 的长度不大于 $\beta$ 的长度，即 $\lvert \alpha \rvert \leq \lvert \beta \rvert$__，则称文法 $G$ 属于 1 型文法。

    - 2 型文法（上下文无关文法）

        设文法 $G = (V_N, V_T, P, S)$，如果任意 $\alpha \rightarrow \beta \in P$，__$\alpha \in V_N$__，则称文法 $G$ 属于 2 型文法。

        - 上下文无关文法的一个显著特征是规则左部有且仅有一个非终结符。

    - 3 型文法（正规文法）

        设文法 $G = (V_N, V_T, P, S)$，如果任意 $\alpha \rightarrow \beta \in P$，$\alpha \in V_N$，__且 $\beta$ 只能是 $aB$ / $Ba$ 或 $a$（除空规则外）__，则称文法 $G$ 属于右 / 左线性 3 型文法。

- 最左推导、最右推导

    若在推导的每一步总是选择当前句型的最左 / 最右边非终结符进行推导，则称这种推导过程为最左 / 最右推导。

    - 最右推导也称规范推导。由规范推导所得句型为规范句型。规范推导的逆过程为规范规约。

- 语法二义性

    如果一个文法 $G$，某个句子存在对应的至少两棵不同的语法树，则称文法 $G$ 是二义性的。

    - 如果文法是无二义性的，一个句子的最左 / 最右推导是唯一的。
    - 文法的二义性不等同于语言的二义性。因为对二义性文法 $G$ 可能存在与之等价的非二义性文法 $G'$。  
        如果一个语言不存在无二义性的文法，则称该语言是先天二义性的。
        - 文法的二义性判定问题是递归不可解的。

- 句型分析

    - 自上而下分析法

        从文法开始符号出发，反复使用规则，寻找匹配符号串（推导）的句型，知道推导出句子或规则用遍。

        - 两个问题
            1. 选择句型中哪一个非终结符进行推导
            2. 选择非终结符的哪一个规则进行推导

    - 自下而上分析法

        从输入符号串 $\alpha$ 开始，逐步进行规约，直至规约出文法开始符号 $S$，否则 $\alpha \notin L(G)$。

        - 通过在句型中寻找 __句柄__ 的途径解决。

- 短语、直接短语与句柄

    设 $G[S]$ 是一文法，$\alpha \beta \delta$ 是文法 $G$ 的句型，如果有 $S \overset{\star}{\Rightarrow} \alpha A \delta$，且 $A \overset{+}{\Rightarrow} \beta$，则称 $\beta$ 是句型 $\alpha \beta \delta$ 的、相对于非终结符 $A$ 的短语。

    特别地，当 $A \Rightarrow \beta$（一步推导）时，又称 $\beta$ 是句型 $\alpha \beta \delta$ 的、相对于非终结符 $A$ 的直接短语（或简单短语）。

    句型的最左直接短语，称为该句型的句柄。

    - 可以根据具体推导、语法树来判断短语、直接短语与句柄。

- 多余规则

    - 有害规则：形如 $U \rightarrow U$ 的规则。
    - 不可达规则：不在任何规则右部出现的非终结符对应的规则。
    - 不可终止规则：从某非终结符开始，不可能推导出任意终结串。

    不含多余规则的文法，称为压缩过的文法。


# 词法分析

词法分析是编译的第一阶段，其任务为从左至右扫描源程序文本，从 __基于字符理解__ 的源程序中分离出符合源程序语言词法的单词，最终转换成 __基于单词理解__ 的源程序。

计算机高级语言一般有关键字、标识符、常数、运算符和界定符。

- 正规文法

- 正规式

    <center>正规式到正规文法转换</center>

    | 正规式 | 正规文法 |
    |:----- |:------ |
    | $A \rightarrow x y$ | $A \rightarrow xB,\ B \rightarrow y$ |
    | $A \rightarrow x* y$ | $A \rightarrow x B,\ A \rightarrow y,\\ B \rightarrow x B,\ B \rightarrow y$ |
    | $A \rightarrow x \vert y$ | $A \rightarrow x,\ A \rightarrow y$ |

    <br />

    <center>正规文法到正规式转换</center>

    | 正规文法 | 正规式 |
    |:------- |:----- |
    | $A \rightarrow xB,\ B \rightarrow y$ | $A \rightarrow x y$ |
    | $A \rightarrow xA \vert y$ | $A \rightarrow x* y$ |
    | $A \rightarrow x,\ A \rightarrow y$ | $A \rightarrow x \vert y$ |

- 有穷自动机（FA）

    - 确定有穷自动机（DFA）

        一个 DFA $M$ 是一个五元组：$M = (K, \Sigma, f, S, Z)$。  
        其中：
        - $K$ 是非空有穷集，每个元素称为状态；
        - $\Sigma$ 是有穷字母表；
        - $f$ 是 $K \times \Sigma \rightarrow K$ 映射，称为状态转换函数；
        - $S \in K$，称为开始状态；
        - $Z \subset K$，称为结束状态集 / 接收状态集。

    - 不确定有穷自动机（NFA）

        一个 NFA $M$ 是一个五元组：$M = (K, \Sigma, f, S, Z)$。  
        与 DFA 不同的是：
        - $f$ 是 $K \times \Sigma \cup \{ \epsilon \} \rightarrow \rho(K)$ 映射，其中 $\rho(K)$ 表示 $K$ 的幂集；
        - $S \subset K$，称为开始状态集。

        - NFA 到 DFA 转换
            1. 置 $K'$ 为k空集；
            2. 计算 $M'$ 的开始状态集 $S' = \epsilon\text{_closure}(S)$，$S'$ 作为 $K'$ 新增状态；
            3. 对于 $K'$ 每一个新增状态 $q$，计算出每一个 $a \in \Sigma$ 的转换状态 $p$，即 $f'(q, a) = p = \epsilon\text{_closure}(M(q, a))$。如果 $p \notin K'$，则 $p$ 作为 $K'$ 的新增状态；
            4. 重复步骤 3，直到 $K'$ 不再出现新增状态为止；
            5. 计算接收状态集 $Z' = \{ q \vert q \in K', q \cap Z \neq \Phi \}$。


- $\epsilon\text{_closure}$

    设 NFA $M = (K, \Sigma, f, S, Z)$，$I \subset K$，则 $\epsilon\text{_closure}(I)$ 计算过程如下：  
    1. $I \subset \epsilon\text{_closure}(I)$
    2. $M(\epsilon\text{_closure}(I), \epsilon) \subset \epsilon\text{_closure}(I)$  
        其中 $M(I, a) = \cup_{q \in I} f(q, a)$。
    3. 重复步骤 2，直至 $\epsilon\text{_closure}(I)$ 不再扩大为止。




