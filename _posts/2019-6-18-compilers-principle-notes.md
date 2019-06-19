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

        设文法 $$G = (V_N, V_T, P, S)$$，如果任意 $$\alpha \rightarrow \beta \in P$$，__$$\alpha$$ 中至少包含一个非终结符__，则称文法 $$G$$ 属于 0 型文法。

    - 1 型文法（上下文有关文法）

        设文法 $$G = (V_N, V_T, P, S)$$，如果任意 $$\alpha \rightarrow \beta \in P$$，$$\alpha$$ 中至少含有一个非终结符，__且除空规则外，$$\alpha$$ 的长度不大于 $$\beta$$ 的长度，即 $$\lvert \alpha \rvert \leq \lvert \beta \rvert$$__，则称文法 $$G$$ 属于 1 型文法。

    - 2 型文法（上下文无关文法）

        设文法 $$G = (V_N, V_T, P, S)$$，如果任意 $$\alpha \rightarrow \beta \in P$$，__$$\alpha \in V_N$$__，则称文法 $$G$$ 属于 2 型文法。

        - 上下文无关文法的一个显著特征是规则左部有且仅有一个非终结符。

    - 3 型文法（正规文法）

        设文法 $$G = (V_N, V_T, P, S)$$，如果任意 $$\alpha \rightarrow \beta \in P$$，$$\alpha \in V_N$$，__且 $$\beta$$ 只能是 $$aB$$ / $$Ba$$ 或 $$a$$（除空规则外）__，则称文法 $$G$$ 属于右 / 左线性 3 型文法。

- 最左推导、最右推导

    若在推导的每一步总是选择当前句型的最左 / 最右边非终结符进行推导，则称这种推导过程为最左 / 最右推导。

    - 最右推导也称规范推导。由规范推导所得句型为规范句型。规范推导的逆过程为规范规约。

- 语法二义性

    如果一个文法 $$G$$，某个句子存在对应的至少两棵不同的语法树，则称文法 $$G$$ 是二义性的。

    - 如果文法是无二义性的，一个句子的最左 / 最右推导是唯一的。
    - 文法的二义性不等同于语言的二义性。因为对二义性文法 $$G$$ 可能存在与之等价的非二义性文法 $$G'$$。  
        如果一个语言不存在无二义性的文法，则称该语言是先天二义性的。
        - 文法的二义性判定问题是递归不可解的。

- 句型分析

    - 自上而下分析法

        从文法开始符号出发，反复使用规则，寻找匹配符号串（推导）的句型，知道推导出句子或规则用遍。

        - 两个问题
            1. 选择句型中哪一个非终结符进行推导
            2. 选择非终结符的哪一个规则进行推导

    - 自下而上分析法

        从输入符号串 $$\alpha$$ 开始，逐步进行规约，直至规约出文法开始符号 $$S$$，否则 $$\alpha \notin L(G)$$。

        - 通过在句型中寻找 __句柄__ 的途径解决。

- 短语、直接短语与句柄

    设 $$G[S]$$ 是一文法，$$\alpha \beta \delta$$ 是文法 $$G$$ 的句型，如果有 $$S \overset{\star}{\Rightarrow} \alpha A \delta$$，且 $$A \overset{+}{\Rightarrow} \beta$$，则称 $$\beta$$ 是句型 $$\alpha \beta \delta$$ 的、相对于非终结符 $$A$$ 的短语。

    特别地，当 $$A \Rightarrow \beta$$（一步推导）时，又称 $$\beta$$ 是句型 $$\alpha \beta \delta$$ 的、相对于非终结符 $$A$$ 的直接短语（或简单短语）。

    句型的最左直接短语，称为该句型的句柄。

    - 可以根据具体推导、语法树来判断短语、直接短语与句柄。

- 多余规则

    - 有害规则：形如 $$U \rightarrow U$$ 的规则。
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
    | $$A \rightarrow x y$$ | $$A \rightarrow xB,\ B \rightarrow y$$ |
    | $$A \rightarrow x* y$$ | $$A \rightarrow x B,\ A \rightarrow y,\\ B \rightarrow x B,\ B \rightarrow y$$ |
    | $$A \rightarrow x \vert y$$ | $$A \rightarrow x,\ A \rightarrow y$$ |

    <br />

    <center>正规文法到正规式转换</center>

    | 正规文法 | 正规式 |
    |:------- |:----- |
    | $$A \rightarrow xB,\ B \rightarrow y$$ | $$A \rightarrow x y$$ |
    | $$A \rightarrow xA \vert y$$ | $$A \rightarrow x* y$$ |
    | $$A \rightarrow x,\ A \rightarrow y$$ | $$A \rightarrow x \vert y$$ |

- 有穷自动机（FA）

    - 确定有穷自动机（DFA）

        一个 DFA $$M$$ 是一个五元组：$$M = (K, \Sigma, f, S, Z)$$。  
        其中：
        - $$K$$ 是非空有穷集，每个元素称为状态；
        - $$\Sigma$$ 是有穷字母表；
        - $$f$$ 是 $$K \times \Sigma \rightarrow K$$ 映射，称为状态转换函数；
        - $$S \in K$$，称为开始状态；
        - $$Z \subset K$$，称为结束状态集 / 接收状态集。

    - 不确定有穷自动机（NFA）

        一个 NFA $$M$$ 是一个五元组：$$M = (K, \Sigma, f, S, Z)$$。  
        与 DFA 不同的是：
        - $$f$$ 是 $$K \times \Sigma \cup \{ \epsilon \} \rightarrow \rho(K)$$ 映射，其中 $$\rho(K)$$ 表示 $$K$$ 的幂集；
        - $$S \subset K$$，称为开始状态集。

    - NFA 到 DFA 转换
        1. 置 $$K'$$ 为k空集；
        2. 计算 $$M'$$ 的开始状态集 $$S' = \epsilon\text{_closure}(S)$$，$$S'$$ 作为 $$K'$$ 新增状态；
        3. 对于 $$K'$$ 每一个新增状态 $$q$$，计算出每一个 $$a \in \Sigma$$ 的转换状态 $$p$$，即 $$f'(q, a) = p = \epsilon\text{_closure}(M(q, a))$$。如果 $$p \notin K'$$，则 $$p$$ 作为 $$K'$$ 的新增状态；
        4. 重复步骤 3，直到 $$K'$$ 不再出现新增状态为止；
        5. 计算接收状态集 $$Z' = \{ q \vert q \in K', q \cap Z \neq \Phi \}$$。

    - DFA 最小化（分割法）
        1. 状态集 $$K$$ 划分为两个状态子集 $$\{ Z, K - Z \}$$，记为 $$\Pi = \{ Z, K - Z \}$$；
        2. 如果 $$\exists I \in \Pi$$，$$\exists a \in \Sigma$$，$$\exists J \in \Pi [ M(I, a) \not\subset J ]$$，即状态子集 $$I \in \Pi$$ 中至少存在两个 $$p$$ 和 $$q$$，使得 $$f(p, a) \in J'$$ 和 $$f(q, a) \in J''$$，且 $$J' \neq J''$$，则将 $$I$$ 分割成 $$I'$$ 和 $$I''$$，即 $$I' = \{ r \vert \forall r \in I [ f(r, a) \in J' ] \}$$，$$I'' = I - J'$$；重置划分 $$\Pi$$：$$\Pi \leftarrow (\Pi - \{ I \}) \cup \{ I', I'' \}$$。
        3. 重复步骤 2，直到满足 $$\forall I \in \Pi$$，$$\forall a \in \Sigma$$，$$\exists J \in \Pi [ M(I, a) \subset J ]$$ 条件为止；
        4. 在 DFA $$M$$ 的基础上，对于划分 $$\Pi$$ 的同一个状态子集中的全部状态及其相应的转换函数合并，最后所得即为最小化的 DFA $$M'$$。

- $$\epsilon\text{_closure}$$

    设 NFA $$M = (K, \Sigma, f, S, Z)$$，$$I \subset K$$，则 $$\epsilon\text{_closure}(I)$$ 计算过程如下：

    1. $$I \subset \epsilon\text{_closure}(I)$$
    2. $$M(\epsilon\text{_closure}(I), \epsilon) \subset \epsilon\text{_closure}(I)$$  
        其中 $$M(I, a) = \cup_{q \in I} f(q, a)$$。
    3. 重复步骤 2，直至 $$\epsilon\text{_closure}(I)$$ 不再扩大为止。

# 自顶向下语法分析

- $$\text{FIRST}$$ 集

    设文法 $$G = (V_N, V_T, P, S)$$，则 $$\text{FIRST}(\alpha) = \{ a \vert \alpha \overset{\star}{\Rightarrow} a \beta, a \in V_T, \alpha, \beta \in V* \}$$  
    特别地，$$\alpha \overset{\star}{\Rightarrow} \epsilon$$，约定 $$\epsilon \in \text{FIRST}(\alpha)$$。

    - 计算方法（$$X \in V_N \cup V_T$$）
        1. 对于所有终结符号 $$X$$，$$\text{FIRST}(X) = \{ X \}$$；
        2. 对于所有空规则 $$X \rightarrow \epsilon$$，$$\text{FIRST}(X) = \{ \epsilon \}$$；
        3. 对于所有形如 $$X \rightarrow a \cdots$$ 的规则，且 $$a \in V_T$$，$$\text{FIRST}(X) \cup = \{ a \}$$；
        4. 对于所有形如 $$X \rightarrow Y_1 Y_2 \cdots Y_3$$ 的规则，  
            若 $$Y_1 \overset{\star}{\Rightarrow} \epsilon$$，$$Y_2 \overset{\star}{\Rightarrow} \epsilon$$，$$\cdots$$，$$Y_{i-1} \overset{\star}{\Rightarrow} \epsilon$$（$$i \leq n$$），则 $$\text{FIRST}(X) \cup = \cup_{j = 1}^{i} \text{FIRST}(Y_j) - \{ \epsilon \}$$；  
            若 $$Y_1 \overset{\star}{\Rightarrow} \epsilon$$，$$Y_2 \overset{\star}{\Rightarrow} \epsilon$$，$$\cdots$$，$$Y_n \overset{\star}{\Rightarrow} \epsilon$$，则 $$\text{FIRST}(X) \cup = \cup_{j = 1}^{n} \text{FIRST}(Y_j) \cup \{ \epsilon \}$$；
        5. 重复步骤 4，直至 $$\text{FIRST}$$ 集不再扩大为止。

- $$\text{FOLLOW}$$ 集

    设文法 $$G = (V_N, V_T, P, S)$$，则 $$\text{FOLLOW}(A) = \{ a \vert S \overset{\star}{\Rightarrow} \alpha A \beta, A \in V_N, a \in \text{FIRST}(\beta), \alpha, \beta \in V* \}$$。  
    （或者 $$\text{FOLLOW}(A) = \{ a \vert S \overset{\star}{\Rightarrow} \cdots A a \cdots, A \in V_N, a \in V_T \}$$）

    - 计算方法（$$X \in V_N$$）
        1. 置 $$\text{FOLLOW}(S) = \{ \# \}$$；
        2. 对所有规则：  
            若 $$A \rightarrow \alpha B \beta$$，且 $$B \in V_N$$，则 $$\text{FOLLOW}(B) \cup = \text{FIRST}(B) - \{ \epsilon \}$$；  
            若 $$\beta \overset{\star}{\Rightarrow} \epsilon$$，则 $$\text{FOLLOW}(B) \cup = \text{FOLLOW}(A)$$；
        3. 重复步骤 2，直至 $$\text{FOLLOW}$$ 集不再扩大为止。

- $$\text{SELECT}$$ 集

    设文法 $$G = (V_N, V_T, P, S)$$，$$A \in V_N$$，$$A \rightarrow \alpha \in P$$，则

    $$
    \text{SELECT}(A \rightarrow \alpha) = \begin{cases}
        &\text{FIRST}(\alpha)  &(\alpha \not\overset{\star}{\Rightarrow} \epsilon) \\
        &(\text{FIRST}(\alpha) - \{ \epsilon \}) \cup \text{FOLLOW}(A)  &(\alpha \overset{\star}{\Rightarrow} \epsilon)
    \end{cases}
    $$

- $$\text{LL}(1)$$ 文法

    文法 $$G$$ 是 $$\text{LL}(1)$$ 的充要条件是文法 $$G$$ 每个 $$U \rightarrow \alpha_1 \vert \alpha_2 \vert \cdots \vert \alpha_n$$ 规则，满足

    $$
    \text{SELECT}(U \rightarrow \alpha_i) \cap \text{SELECT}(U \rightarrow \alpha_j) = \Phi \\
    (i \neq j, 1 \leq i \leq n, 1 \leq j \leq n)
    $$

    - 确定的自顶向下语法分析不必穷举推导过程，避免了回溯现象，也称不带回溯的自顶向下语法分析。
        - 递归子程序法

            将每个非终结符编写为一个递归子程序，子程序中将输入串下一个符号逐个与选择集进行判定，当下一个符号不属于任何选择集时报错。

        - 预测分析法
            - 输入栈、分析栈、分析表
            - 分析栈初始内容为：$$\# \ S$$
            - 输入栈初始内容为：$$a \ b \ c \ \cdots \ \#$$
            - 查看输入栈栈顶元素，将分析栈栈顶元素规约为分析表中对应规则右部。
            - 若分析栈栈顶与输入栈栈顶的终结符匹配，则同时弹出。
            - 直至分析栈与输入栈中仅含界符时终止。
    - $$\text{LL}(1)$$ 文法的判定
        1. 计算可推导出 $$\epsilon$$ 的非终结符号；
        2. 计算非终结符号的 $$\text{FIRST}(X)$$ 集；
        3. 计算规则右部的 $$\text{FIRST}(\alpha)$$ 集；
        4. 计算非终结符号的 $$\text{FOLLOW}(A)$$ 集；
        5. 计算规则的 $$\text{SELECT}$$ 集。
    - 某些非 $$\text{LL}(1)$$ 文法到 $$\text{LL}(1)$$ 文法的等价变换
        - 提取左公因子法

            $$
            \begin{align}
            &A \rightarrow \alpha \beta_1 \vert \alpha \beta_2 \vert \cdots \vert \alpha \beta_n \vert \gamma_1 \vert \cdots \vert \gamma_m \\\\
            \Rightarrow &A \rightarrow \alpha B \vert \gamma_1 \vert \cdots \vert \gamma_m \\
            &B \rightarrow \beta_1 \vert \beta_2 \vert \cdots \vert \beta_n
            \end{align}
            $$

        - 消除左递归法
            - 消除直接左递归

                $$
                \begin{align}
                &A \rightarrow A \alpha_1 \vert A \alpha_2 \vert \cdots \vert A \alpha_n \vert \beta_1 \vert \cdots \vert \beta_m \\\\
                \Rightarrow &A \rightarrow \beta_1 A' \vert \cdots \vert \beta_m A' \\
                &A' \rightarrow \alpha_1 A' \vert \cdots \vert \alpha_n A' \vert \epsilon
                \end{align}
                $$

            - 消除间接左递归
                - 代入原规则，消除直接左递归。
    - 出错处理
        - 跳过输入栈中符号，直到遇到 $$\text{FOLLOW}(A)$$ / $$\text{FIRST}(A)$$ 中符号；

# 自底向上优先分析

- 简单优先文法

    - 简单优先关系
        - $$X \doteq Y \ \text{iff} \ A \rightarrow \cdots X Y \cdots \in P$$
        - $$X \lessdot Y \ \text{iff} \ A \rightarrow \cdots X B \cdots \in P, B \overset{+}{\Rightarrow} Y \cdots$$
        - $$X \gtrdot Y \ \text{iff} \ A \rightarrow \cdots B D \cdots \in P, B \overset{+}{\Rightarrow} \cdots X, D \overset{\star}{\Rightarrow} Y \cdots$$

    设 $$G$$ 为简单优先文法，句型 $$\# a_1 a_2 \cdots a_{i-1} a_i a_{i+1} \cdots a_{j-1} a_j a_{j+1} \cdots a_n \#$$，且存在下列关系：

    $$
    a_{i-1} \lessdot a_i a_{i+1} \cdots a_{j-1} a_j \gtrdot a_{j+1} \ ,
    $$

    则字串 $$a_i \cdots a_j$$ 是句型的直接短语。特别地，如果这个字串是句型最左字串，则该字串是该句型的句柄。

    不断用句柄对应规则的右部进行规约，直接规约出句型 $$\# S \# \leftarrow S'$$（增加规则）为止。

- 算符优先文法

    - 算符优先关系

        与简单优先分析法不同，算符优先分析法仅利用相邻两个终结符之间优先关系寻找归约字串。这个归约字串不是句柄，是一种特殊的直接短语，“最左素短语”。

        - $$a \doteq b \ \text{iff} \ A \rightarrow \cdots a b \cdots \in P \ \text{or} \ A \rightarrow \cdots a B b \cdots \in P$$
        - $$a \lessdot b \ \text{iff} \ A \rightarrow \cdots a B \cdots \in P, B \overset{+}{\Rightarrow} b \cdots \ \text{or} \ B \overset{+}{\Rightarrow} C b \cdots$$
        - $$a \gtrdot b \ \text{iff} \ A \rightarrow \cdots B b \cdots \in P, B \overset{+}{\Rightarrow} \cdots a \ \text{or} \ B \overset{+}{\Rightarrow} \cdots C a$$

    若 $$G$$ 中没有形如 $$A \rightarrow \cdots B C \cdots$$ 的规则，则称文法 $$G$$ 为算符文法（Operator Grammar）。

    若任意两终结符之间至多存在 $$\doteq$$、$$lessdot$$、$$gtrdot$$ 三种算符优先关系之一，则称文法 $$G$$ 为算符优先文法（Operator Precedence Grammar）。

    句型的素短语是满足下列两个条件的短语：

    1. 至少含有一个终结符；
    2. 除自身之外不再包含其他素短语。

    特别地，最左边的素短语为最左素短语。

    - $$\text{FIRSTVT}(B) = \{ b \vert B \overset{+}{\Rightarrow} b \cdots \ \text{or} \ B \overset{+}{\Rightarrow} C b \cdots \}$$
        - 计算方法
            1. 对形如 $$A \rightarrow a \cdots$$、$$A \rightarrow B a \cdots$$ 的规则，$$\text{FIRSTVT}(A) \cup = \{ a \}$$；
            2. 对形如 $$A \rightarrow B \cdots$$ 的规则，$$\text{FIRSTVT}(A) \cup = \text{FIRSTVT}(B)$$；
            3. 重复步骤 2，直至 $$\text{FIRSTVT}$$ 集不再扩大为止。
    - $$\text{LASTVT}(B) = \{ a \vert B \overset{+}{\Rightarrow} \cdots a \ \text{or} \ B \overset{+}{\Rightarrow} \cdots a C \}$$
        - 计算方法
            1. 对形如 $$A \rightarrow \cdots a$$、$$A \rightarrow \cdots a B$$ 的规则，$$\text{LASTVT}(A) \cup = \{ a \}$$；
            2. 对形如 $$A \rightarrow \cdots B$$ 的规则，$$\text{LASTVT}(A) \cup = \text{LASTVT}(B)$$；
            3. 重复步骤 2，直至 $$\text{LASTVT}$$ 集不再扩大为止。
    - 算符优先关系计算方法
        - 若 $$A \rightarrow \cdots a b \cdots$$ 或 $$A \rightarrow \cdots a B b \cdots$$，则 $$a \doteq b$$；
        - 若 $$A \rightarrow \cdots a B \cdots$$，且 $$b \in \text{FIRSTVT}(B)$$，则 $$a \lessdot b$$；
        - 若 $$A \rightarrow \cdots B a \cdots$$，且 $$b \in \text{LASTVT}(B)$$，则 $$a \gtrdot b$$；

    - 算符优先函数
        1. 对 $$a \in V_T$$，$$f(a) = g(a) = 1$$
        2. 对每个关系对偶 $$\langle a, b \rangle$$：
            - 若 $$a \doteq b,\ f(a) \neq g(b)$$，则 $$\min \{ f(a), g(b) \} = \max \{ f(a), g(b) \}$$；
            - 若 $$a \lessdot b,\ f(a) \geq g(b)$$，则 $$g(b) = f(a) + 1$$；
            - 若 $$a \gtrdot b,\ f(a) \leq g(b)$$，则 $$f(a) = g(b) + 1$$。
        3. 重复步骤 2，直至过程收敛，或有函数值大于 $$2 \lvert V_T \rvert$$，则函数不存在。

# $$\text{LR}$$ 分析

$$\text{LR}(K)$$ 分析法。
- $$\text{L}$$：从左至右扫描字符串；
- $$\text{R}$$：最右推导逆过程，即规范规约；
- $$K$$：向右查看输入串符号个数。

<br />

- 输入栈 $$I$$（栈顶元素为 $$a$$）
- 分析栈：状态栈 $$S.Q$$（栈顶元素为 $$q$$）、符号栈 $$S.X$$
- 分析表：$$M : \text{ACTION}, \text{GOTO}$$
- 总控程序
    1. 初始化：$$(0, \#)$$ 进栈 $$S$$；
    2. 移进（__s__hift）：如果 $$M.\text{ACTION}[q, a] = s_j$$，则将 $$(j, a)$$ 进栈 $$S$$，输入栈 $$I$$ 出栈，转步骤 2；
    3. 归约（__r__educe）：如果 $$M.\text{ACTION}[q, a] = r_i$$，则  
        1. 令第 $$i$$ 条规则为 $$A \rightarrow \alpha$$，将 $$\lvert \alpha \rvert$$ 个状态和符号退出分析栈 $$S$$；
        2. 令 $$q'$$ 为此刻状态栈 $$S.Q$$ 栈顶元素。若 $$M.\text{GOTO}[q', A] = j$$，将 $$(j, A)$$ 进栈 $$S$$，转步骤 2；否则 $$M.\text{GOTO}[q', A] = e_k$$，转出错处理 $$\text{ERROR}()$$。
    4. 报错：如果 $$M.\text{ACTION}[q, a] = e_k$$，转出错处理 $$\text{ERROR}()$$；
    5. 接收：如果 $$M.\text{ACTION}[q, a] = \text{acc}$$，则输出 $$\text{OK}$$，结束。

<br />

- $$\text{LR}(0)$$ 分析
    - 可归前缀和活前缀
        - 将符号串的任意含有头符号的子串称为前缀。特别地，空串 $$\epsilon$$ 为任意串的前缀。
        - 若 $$S \underset{R}{\overset{\star}{\Rightarrow}} \alpha A \omega \underset{R}{\Rightarrow} \alpha \beta \omega$$ 是句型 $$\alpha \beta \omega$$ 的规范推导，则 $$\alpha \beta$$ 为可归前缀，$$\alpha \beta$$ 的前缀为活前缀。

        > 尾符号恰好是句柄 $$\beta$$ 尾符号的文法规范句型的前缀，称为可归前缀，可归前缀的前缀称为活前缀。

    - $$\text{LR}(0)$$ 项目
        - 移进项目：$$A \rightarrow \alpha \cdot a \beta$$
        - 待约项目：$$A \rightarrow \alpha \cdot X \beta$$
        - 归约项目：$$A \rightarrow \alpha \cdot$$
        - 接受项目：$$S' \rightarrow \alpha \cdot$$

    - $$\text{Move}(I, X) = \{ A \rightarrow \alpha X \cdot \beta \vert A \rightarrow \alpha \cdot X \beta \in I \}$$
    - $$\text{closure}(I)$$
        1. $$I \subset \text{closure}(I)$$
        2. $$\{ B \rightarrow \cdot \gamma \vert A \rightarrow \alpha \cdot B \beta \in \text{closure}(I) \} \subset \text{closure}(I)$$
        3. 重复步骤 2，直至 $$\text{closure}(I)$$ 不再扩大为止。

    - $$\text{LR}(0)$$ 识别活前缀的 DFA $$M = (K, \Sigma, f, S, Z)$$ 的构造方法
        1. $$K \subseteq \rho$$（$$\text{LR}(0)$$ 项目集规范族）
        2. $$\Sigma = V_N \cup V_T$$
        3. $$f(I, X) = \text{closure}(\text{Move}(I, X)),\ I \in K, X \in \Sigma$$
        4. $$S = \text{closure}(S' \rightarrow \cdot S)$$
        5. $$Z = \{ q \vert q \in K, q \ \text{含归约项目} \}$$

    - $$\text{LR}(0)$$ 分析表的构造
        1. 对每一个 $$\text{LR}(0)$$ 项目
            - 若为移进项目 $$A \rightarrow \alpha \cdot a \beta \in I_k,\ f(I_k, a) = I_j$$，则 $$M.\text{ACTION}[k, a] = s_j$$；
            - 若为归约项目 $$A \rightarrow \alpha \cdot \in I_k$$，规则 $$A \rightarrow \alpha$$ 标号为 $$i$$，则对 $$\forall a \in V_T \cup \{ \# \}$$，$$M.\text{ACTION}[k, a] = r_i$$；
            - 若为接受项目 $$S' \rightarrow S \cdot \in I_k$$，则 $$M.\text{ACTION}[k, \#] = \text{acc}$$；
            - 另外，若 $$f(I_k, A) = I_j,\ A \in V_N$$，则 $$M.\text{GOTO}[k, A] = j$$。
        2. 其余空位置 $$e_{\cdot}$$。
















