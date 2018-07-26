---
layout: article
title: CS294 Deep Reinforcement Learning Notes
tags: MachineLearning DeepLearning ReinforcementLearning
---

{:toc}

---

# Lectures

## The "REPL"

### The Roles

-   Given by **Agent**

    $$
    \pi_\theta(\mathbf{u}_t \vert \mathbf{o}_t) \,, \mathbf{o}_t \rightarrow \mathbf{x}_t
    $$

    -   $$\pi$$ as *policy*, $$\theta$$ as parameters in policy model

        >   **Note**  
        >   Action space could be discrete, consider using **Monte-Carlo Tree Search** then.  

    -   $$\mathbf{u}$$, $$\mathbf{x}$$, $$\mathbf{o}$$ as *action*, *ground-truth state* and *observed state* respectively, $$t$$ for current time step

-   Given by **Environment**

    $$
    \mathrm{p}(\mathbf{x}_{t+1} \vert \mathbf{x}_t, \mathbf{u}_t)
    $$

    -   $$\mathrm{p}$$ (stocastically) or $$f$$ for set of state transition rules built within the environment

-   Defined by **User**

    $$
    r(\mathbf{x}, \mathbf{u})
    $$

    -   $$r$$ for *reward*, or $$c := -r$$ for *cost*

>   **Note**  
>   Clever audiences may have found the **3 Places to Apply NN** in a RL model by now.  
>   -   **Agent:** learn Policies
>   -   **Environment:** learn Dynamics
>   -   **Feedback:** learn Rewards


### The Goal

With constraints (collocation method), will be

$$
\min_{\mathbf{u}_1, \dots, \mathbf{u}_T, \mathbf{x}_1, \dots, \mathbf{x}_T} \sum_{t=1}^{T} = c(\mathbf{x}_t, \mathbf{u}_t) \text{ such that } \mathbf{x}_t = f(\mathbf{x}_{t-1}, \mathbf{u}_{t-1}) \hspace{1em}\cdots\cdots\cdots\cdots \, (1)
$$

---

## LQR Algorithm

LQR, or Linear Quadratic Regulator.  

**Goal:** Solve $$\text{Eq. }(1)$$ or expanded as

$$
\min_{\mathbf{u}_1, \dots, \mathbf{u}_T, \mathbf{x}_1, \dots, \mathbf{x}_T} = c(\mathbf{x}_1, \mathbf{u}_2) + c(f(\mathbf{x}_1, \mathbf{u}_1), \mathbf{u}_2) + \cdots + c(f(f(\dots)\dots), \mathbf{u}_T) \hspace{1em}\cdots\cdots\cdots\cdots \, (1')
$$

Here presents an example with known *linear dynamics* and *linear quadratic cost* (i.e. all $$\mathrm{C}$$ and $$\mathrm{F}$$ can be substituted)  

### Linear Dynamics System

$$
f(\mathbf{x}_t, \mathbf{u}_t) = \mathbf{F}_t \begin{bmatrix} \mathbf{x}_t \\ \mathbf{u}_t \end{bmatrix} + \mathbf{f}_t \hspace{1em}\cdots\cdots\cdots\cdots \, (2)
$$

>   **Note**  
>   $$\mathbf{x}$$ may contain quadraic terms of its elements, velocity $$\dot{x}$$ and acceleration $$\ddot{x}$$ for example.  

### Linear Quadratic Cost

$$
c(\mathbf{x}_t, \mathbf{u}_t) = \frac{1}{2} \begin{bmatrix} \mathbf{x}_t \\ \mathbf{u}_t \end{bmatrix}^T \mathbf{C}_t \begin{bmatrix} \mathbf{x}_t \\ \mathbf{u}_t \end{bmatrix} + \begin{bmatrix} \mathbf{x}_t \\ \mathbf{u}_t \end{bmatrix}^T \mathbf{c}_t \hspace{1em}\cdots\cdots\cdots\cdots \, (3)
$$

**Methodology**  

Solve symbols backwards (from last action $$\mathbf{u}_T$$ to first state $$\mathbf{x}_1$$), fill values forwards (from first action $$\mathbf{u}_1$$ to last state $$\mathbf{x}_T$$).  

$$
\begin{align}
&\text{/* 1. Backward recursion */} \\
&\text{for } t = T \text{ to } 1 \text{:} \\
&\hspace{1em} \mathbf{Q}_t = \mathbf{C}_t + \mathbf{F}_t^T \mathbf{V}_{t+1} \mathbf{F}_t \\
&\hspace{1em} \mathbf{q}_t = \mathbf{c}_t + \mathbf{F}_t^T \mathbf{V}_{t+1} \mathbf{f}_t + \mathbf{F}_t^T \mathbf{v}_{t+1} \\
&\hspace{1em} Q(\mathbf{x}_t, \mathbf{u}_t) = \mathrm{const} + \frac{1}{2} \begin{bmatrix} \mathbf{x}_t \\ \mathbf{u}_t \end{bmatrix}^T \mathbf{Q}_t \begin{bmatrix} \mathbf{x}_t \\ \mathbf{u}_t \end{bmatrix} + \begin{bmatrix} \mathbf{x}_t \\ \mathbf{u}_t \end{bmatrix}^T \mathbf{q}_t \hspace{2em} \text{(total cost from now until end if take } \mathbf{u}_t \text{ from state } \mathbf{x}_t \text{)} \\
&\hspace{1em} \mathbf{u}_t \leftarrow \arg \min_{\mathbf{u}_t} Q(\mathbf{x}_t, \mathbf{u}_t) = \mathbf{K}_t \mathbf{x}_t + \mathbf{k}_t \\
&\hspace{1em} \mathbf{K}_t = -\mathbf{Q}_{\mathbf{u}_t, \mathbf{u}_t}^{-1} \mathbf{Q}_{\mathbf{u}_t, \mathbf{x}_t} \\
&\hspace{1em} \mathbf{k}_t = -\mathbf{Q}_{\mathbf{u}_t, \mathbf{u}_t}^{-1} \mathbf{q}_{\mathbf{u}_t} \\
&\hspace{1em} \mathbf{V}_t = \mathbf{Q}_{\mathbf{x}_t, \mathbf{x}_t} + \mathbf{Q}_{\mathbf{x}_t, \mathbf{u}_t} \mathbf{K}_t + \mathbf{K}_t^T \mathbf{Q}_{\mathbf{u}_t, \mathbf{x}_t} + \mathbf{K}_t^T \mathbf{Q}_{\mathbf{u}_t, \mathbf{u}_t} \mathbf{K}_t \\
&\hspace{1em} \mathbf{v}_t = \mathbf{q}_{\mathbf{x}_t} + \mathbf{Q}_{\mathbf{x}_t, \mathbf{u}_t} \mathbf{k}_t + \mathbf{K}_t^T \mathbf{Q}_{\mathbf{u}_t} + \mathbf{K}_t^T \mathbf{Q}_{\mathbf{u}_t, \mathbf{u}_t} \mathbf{k}_t \\
&\hspace{1em} V(\mathbf{x}_t) = \mathrm{const} + \frac{1}{2} \mathbf{x}_t^T \mathbf{V}_t \mathbf{x}_t + \mathbf{x}_t^T \mathbf{v}_t \hspace{2em} \text{(total cost from now until end from state } \mathbf{x}_t \text{)} \\
&\\
&\because \, \mathbf{x}_1 \text{ is given} \\
&\\
&\text{/* 2. Forward recursion */} \\
&\text{for } t = 1 \text{ to } T \text{:} \\
&\hspace{1em} \mathbf{u}_t = \mathbf{K}_t \mathbf{x}_t + \mathbf{k}_t \\
&\hspace{1em} \mathbf{x}_{t+1} = f(\mathbf{x}_t, \mathbf{u}_t)
\end{align}
\\
$$

>   **Flags**  
>   - [x] You'd want to touch this thing no more
>   - [ ] It can be solved properly and easily with an NN (and escape from Newton's Method for good)

### Nonlinear Case

Approximate linear-quadratic case using Taylor expansion.  

-   differential dynamic programming (or DDP)
-   iterative LQR
    -   for all reference points, run LQR backward recursion on approximation, then run forward recursion on true nonlinear system (with hacks that will not be discussed on this post)

---

## Learn the Model / Dynamics

**Goal**  

Discover $$p(\mathbf{x}_{t+1} \vert \mathbf{x}_t, \mathbf{u}_t)$$.  

### Model-Based RL v1.5

1.  Explore the playground with a particular goal
    -   run base policy $$\pi_0(\mathbf{u}_t \vert \mathbf{x}_t)$$ to collect $$\mathcal{D}=\{ (\mathbf{x}, \mathbf{u}, \mathbf{x}')_i \}$$
2.  Learn from exprerience to generate $$p_{\pi_0}$$, the observed model
    -   learn dynamics model $$f(\mathbf{x}, \mathbf{u})$$ to minimize $$\sum_i \lVert f(\mathbf{x}_i, \mathbf{u}_i) - \mathbf{x}_i' \rVert ^2$$
3.  Dream of more adventures
    -   backpropagate through $$f(\mathbf{x}, \mathbf{u})$$ to choose actions
4.  Take real actions, **verify the learnt exprerience**, i.e. test observed model $$p_{\pi_0}$$ against ground-truth $$p_{\pi_f}$$ to find if any mismatch exists, and try to get even close to our goal
    -   [v1.5] (MPC) execute those actions and observe consequent states $$\mathbf{x}'$$
    -   (DAgger) append the resulting data $$\{ (\mathbf{x}, \mathbf{u}, \mathbf{x}')_j \}$$ to $$\mathcal{D}$$
    -   [v1.5] back to step 3 to **re-plan** actions (correct errors before going off road, planning algorithm required), loop for N times (or N-parallelism)
    -   back to step 2

### Model-Based RL v2.0

1.  Explore the playground with a particular goal
    -   run base policy $$\pi_0(\mathbf{u}_t \vert \mathbf{x}_t)$$ to collect $$\mathcal{D}=\{ (\mathbf{x}, \mathbf{u}, \mathbf{x}')_i \}$$
2.  Learn from exprerience to generate $$p_{\pi_0}$$, the observed model
    -   learn dynamics model $$f(\mathbf{x}, \mathbf{u})$$ to minimize $$\sum_i \lVert f(\mathbf{x}_i, \mathbf{u}_i) - \mathbf{x}_i' \rVert ^2$$
3.  Dream of more adventures
    -   backpropagate through $$f(\mathbf{x}, \mathbf{u})$$ into the policy (along the **whole** decision chain) to optimze $$\pi_\theta(\mathbf{u}_t \vert \mathbf{x}_t)$$
    -   run $$\pi_\theta(\mathbf{u}_t \vert \mathbf{x}_t)$$
4.  Take real actions, and try to get even close to our goal
    -   (DAgger) append the resulting data $$\{ (\mathbf{x}, \mathbf{u}, \mathbf{x}')_j \}$$ to $$\mathcal{D}$$
    -   back to step 2

---

## Train Policies (Model-Based RL)

-   No need to replan
-   Potentially better generalization

**Goal**  

$$
\min_{\tau, \theta} c(\tau) \text{ such that } \mathbf{u}_t = \pi_\theta(\mathbf{x}_t)
$$

### Dual Gradient Descent (Augmented Lagrangian)

$$
\bar{\mathcal{L}}(\mathbf{x}, \lambda) = f(\mathbf{x}) + \lambda C(\mathbf{x}) + \rho \lVert C(\mathbf{x}) \rVert ^2
$$

Where the last squared error term is the *augmented* part.  

1.  Find $$\mathbf{x}^\star \leftarrow \arg \min_\mathbf{x} \bar{\mathcal{L}}(\mathbf{x}, \lambda)$$
2.  Compute $$\frac{dg}{d\lambda} = \frac{d\bar{\mathcal{L}}}{d\lambda}(\mathbf{x}^\star, \lambda)$$
3.  $$\lambda \leftarrow \lambda + \alpha \frac{dg}{d\lambda}$$

So here by now,

$$
\bar{\mathcal{L}}(\tau, \theta, \lambda) = c(\tau) + \sum_{t=1}^T \lambda_t (\pi_\theta(\mathbf{x}_t) - \mathbf{u}_t) + \sum_{t=1}^T \rho_t (\pi_\theta(\mathbf{x}_t) - \mathbf{u}_t)^2
$$

1.  Find $$\tau \leftarrow \arg \min_\tau \bar{\mathcal{L}}(\tau, \theta, \lambda)$$
    -   find a good trajectory
    -   Optimize $$p(\tau)$$ w.r.t. some surrogate $$\tilde{c}(\mathbf{x}_t, \mathbf{u}_t)$$
    -   sequential, may use iLQR
2.  Find $$\theta \leftarrow \arg \min_\theta \bar{\mathcal{L}}(\tau, \theta, \lambda)$$
    -   learn (imitational) the policy upon the trajectory
    -   Optimize $$\theta$$ w.r.t. some supervised objective
    -   non-sequential, may use SGD
3.  $$\lambda \leftarrow \lambda + \alpha \frac{dg}{d\lambda}$$
    -   do gradient descent
    -   Increment or modify dual variables $$\lambda$$

### Imitating MPC: PLATO Algorithm

1.  train $$\pi_\theta(\mathbf{u}_t \vert \mathbf{o}_t)$$ from human data $$\mathcal{D} = \{ \mathbf{o}_1, \mathbf{u}_1, \dots, \mathbf{o}_N, \mathbf{u}_N \}$$
2.  run (in real dynamics, e.g. real world) $$\hat{\pi}(\mathbf{u}_t \vert \mathbf{o}_t)$$ to get dataset $$\mathcal{D} = \{ \mathbf{o}_1, \dots, \mathbf{o}_M \}$$
    -   $$\hat{\pi}(\mathbf{u}_t \vert \mathbf{x}_t) = \arg \min_\hat{\pi} \sum_{t' = t}^T E_\hat{\pi} [ c(\mathbf{x}_{t'}, \mathbf{u}_{t'}) ] + \lambda D_{\mathrm{KL}}(\hat{\pi}(\mathbf{u}_t \vert \mathbf{x}_t) \Vert \pi_\theta(\mathbf{u}_t \vert \mathbf{x}_t))$$
        -   keep newly generated training trajectories different from but relatively **close** to original ones (could be human interfering manually via MPC), in order to avoid financially unacceptabe training cost, for it may lead to physical damage on the test object / environment (e.g. autopilot drone bump into a tree, the model will be taught a good lesson, but the drone is broken for good)
3.  Ask computer to label $$\mathcal{D}_\pi$$ with actions $$\mathbf{u}_t$$
4.  (DAgger) $$\mathcal{D} \leftarrow \mathcal{D} \cup \mathcal{D}_\pi$$

---

## Inverse Dynamics Model

$$\arg\min$$ the dynamics equations.  

### Model Representation

**Dynamics equation**  

$$
M(\mathbf{x}) \ddot{\mathbf{x}} + C(\mathbf{x}, \dot{\mathbf{x}}) \dot{\mathbf{x}} = B \mathbf{u} + J(\mathbf{x})^T \mathbf{f}
$$

**Inverse dynamics function**  

$$
f^{-1}(\mathbf{x}_{t-1}, \mathbf{x}_{t}, \mathbf{x}_{t+1}) = \arg \min_{\mathbf{u}, \mathbf{f}} \lVert \text{ difference of dynamics equation } \rVert ^2
$$

**Inverse dynamics residual**  

Likelihood of being on the trajectory $$(\mathbf{x}_{t-1}, \mathbf{x}_{t}, \mathbf{x}_{t+1})$$.  

$$
r^{-1}(\mathbf{x}_{t-1}, \mathbf{x}_{t}, \mathbf{x}_{t+1}) = \min_{\mathbf{u}, \mathbf{f}} \lVert \text{ difference of dynamics equation } \rVert ^2
$$

### Forward Shooting / Direct Collocation Method

**Forward Shooting**  

$$
\min_{\mathbf{u}_1, \dots, \mathbf{u}_T} \sum_t c_t(\mathbf{x}_t) \text{ s.t. } \mathbf{x}_{t+1} = f(\mathbf{x}_t, \mathbf{u}_t)
$$

-   Optimize over controls
-   State trajectory is implicit
-   Dynamics is an implicit constraint (always satisfied)

**Direct Collocation**  

$$
\min_{\mathbf{x}_1, \dots, \mathbf{x}_T} \sum_t c_t(\mathbf{x}_t) \text{ s.t. } f^{-1}(\mathbf{x}_t, \mathbf{x}_{t+1}) = \mathbf{u}_t \in \mathcal{U}
$$

-   Optimize over states
-   Controls and forces are implicit
-   Dynamics is explict constraint (can be made hard or soft)

### Dynamics with Contact

**Problems**  

-   Discontinuous jumps in contact forces (and their number)
-   No gradient information from inactive contacts

**Contact-Invariant Optimization**  

-   $$c_{t, n} = 1$$, the controller $$n$$ must be touching something and not sliding
-   $$c_{t, n} = 0$$, the controller $$n$$ is unconstrained

**Dynamics Consistency**  

Assume all forces are active (contact set is constant), but apply high penalties for use of forces where $$c = 0$$.  

$$
f^{-1}(\mathbf{x}_{t-1}, \mathbf{x}_{t}, \mathbf{x}_{t+1}) = \arg \min_{\mathbf{u}, \mathbf{f}} \lVert \text{ difference of dynamics equation } \rVert ^2 + \sum_{i} \lVert \mathbf{f}_i \rVert ^2 / (c_i + \epsilon)
$$

### Policy Drifting

Supervised learning in RL is different from traditional supervised learning, like in recognition tasks, in the way that the *steps* are now no longer independent of time / sequence with each other. Errors in feed-forward results will accumulate, getting the model into states that are off the labeled trajectory.  

**Solution: Noisy Training Data**  

Simulate expert policies at states that are slightly off, by generating a correction term pointing towards the opposite direction of the error.  

-   input: $$\mathbf{x} + \epsilon$$
-   output: $$\mathbf{u} + \mathbf{K} \epsilon$$

### Decompose Policy Optimization Problem

-   trajectory optimization
    -   stay close to policy

    $$
    \min_{\mathbf{x}} \sum_t C(\mathbf{x}_t) + \lVert \pi_\theta(\mathbf{x}_t) - \mathbf{u}_t \rVert ^2
    $$

- regression

  $$
  \min_\theta \sum_{i, t} \lVert \pi_\theta(\mathbf{x}_{i, t}) - \mathbf{u}_{i, t} \rVert ^2
  $$





### Movement with Model Uncertianty

-   Generate noisy models varying:
    -   limb mass
    -   limb conter of mass
    -   contact locations
    -   etc.
-   Optimize over multiple state trajectories
-   Single control trajectory
-   Execute control trajectory in open loop

### Interactive Policies with Online Model Learning

-   Offline
    -   Train policy to output desired next state $$\bar{\mathbf{x}}_{t+1}$$
-   At every timestep
    -   Learn robot dynamics on the fly from past observations $$\mathbf{x}_{t+1} = f(\mathbf{x}_t, \mathbf{u}_t)$$
    -   Query policy for $$\bar{\mathbf{x}}_{t+1}$$
    -   Solve for robot torques $$\mathbf{u}^*$$ such that $$\bar{\mathbf{x}}_{t+1} = f(\mathbf{x}_t, \mathbf{u}^*)$$

---

## Markov Decision Processes and Solving Finite Problems

### Markov Decision Process

Defined by the following components:  
-   $$\mathcal{S}$$: **state space**
-   $$\mathcal{A}$$: **action space**
-   $$P(r, s' \vert s, a)$$: a transition probability distribution
    -   $$r$$: reward
    -   $$s$$, $$s'$$: state and next state

### Partially Observed MDPs

-   Instead of obsrving full state $$s$$, agent observes $$y$$ (previously denoted as $$o$$), with $$y \sim P(y \vert s)$$
-   A MDP can be trivially mapped onto a POMDP
- A POMBDP can be mapped on to an MDP

  $$
  \tilde{s}_0 = \{ y_0 \} ,\, \tilde{s}_1 = \{ y_0, y_1 \} ,\, \tilde{s}_2 = \{ y_0, y_1, y_2 \} ,\, \dots
  $$

### Problems Involving MDPs

-   **Policy optimization**: maximize expected reward w.r.t. policy $$\pi$$

    $$
    \max_\pi \mathbb{E} \bigg[ \sum_{t=0}^\infty r_t \bigg]
    $$

-   **Policy evaluation**: compute exprected return for fixed policy $$\pi$$
    -   return := sum of future rewards in an episode, i.e. a trajectory
        -   Discounted return: $$r_t + \gamma r_{t+1} + \gamma^2 r_{t+2} + \cdots$$
        -   Undiscounted return: $$r_t + r_{t+1} + \cdots + r_{T-1} + V(s_T)$$
    -   Performance of policy: $$ \eta(\pi) = \mathbb{E} [ \sum_{t=0}^\infty \gamma^t r_t ] $$
    -   State value function: $$ V^\pi(s) = \mathbb{E} [ \sum_{t=0}^\infty \gamma^t r_t \vert s_0 = s ] $$
    -   State-action value function: $$ Q^\pi(s, a) = \mathbb{E} [ \sum_{t=0}^\infty \gamma^t r_t \vert s_0 = s, a_0 = a ] $$

#### Policy Evaluation

##### Value Iteration: Finite Horizon Case

_General idea of LQR algorithm._  

-   Problem

    $$
    \min_{\pi_0, \dots, \pi_{T-1}} \mathbb{E}[ r_0 + r_1 + \cdots + r_{T-1} + V_T(s_T) ]
    $$

-   Swap maxes and exprectations

    $$
    \max_{\pi_0} \mathbb{E} \bigg[ r_0 + \max_{\pi_1} \mathbb{E} \big[ r_1 + \cdots + \max_{\pi_{T-1}} \mathbb{E} [ r_{T-1} + V_T(s_T) ] \big] \bigg]
    $$

-   Solve innermost problem

    $$
    \begin{align}
    &\text{for each } s \in \mathcal{S} \text{ :} \\
    &\hspace{1em} \pi_{T-1}(s), V_{T-1}(s) = \underset{a}{\text{maximize}} \, \mathbb{E}_{s_T} [ r_{T-1} + V_T(s_T) ]
    \end{align}
    $$

- Substitude original problem recurrently given

  $$
  V_{T-1} = \max_{\pi_{T-1}} \mathbb{E} [ r_{T-1} + V_T(s_T) ]
  $$

##### Discounted Setting

_Dealing with infinite, or large, timestep._  

-   Discount factor $$\gamma \in [0, 1)$$, downweights future rewards
-   Discounted return: $$r_0 + \gamma r_1 + \gamma^2 r_2 + \cdots$$
-   Effective time horizon $$(1 + \gamma + \gamma^2 + \cdots) = 1 / (1 - \gamma)$$
-   Discounted problem can be obtained by adding transitions to *sink state*, where agent gets stuck and recieves zero reward

    $$
    \tilde{P}(s' \vert s, a) = \begin{cases}
    P(s' \vert s, a) \text{ with probability } \gamma \\
    \text{sink state with probability } 1 - \gamma
    \end{cases}
    $$

##### Infinite-Horizon V.I. Via Finite-Horizon V.I.

-   Pretend there exists finite horizon $$T$$, ignore $$r_T, r_{T+1}, \dots$$
    -   error $$\epsilon \leq r_\max \gamma^T / (1 - \gamma)$$
    -   resulting nonstationary policy only suboptimal by $$\epsilon$$
    -   $$\pi_0, V_0$$ converges to optimal policy as $$T \rightarrow \infty$$

##### Infinite-Horizon V.I. Via Operator View

-   $$V \in \mathbb{R}^{\#\mathcal{S}}$$: $$V$$ is a vector with number of dimensions equal to the degree of freedomness of state
-   V.I. update is a function $$\mathcal{T}: \mathbb{R}^{\#\mathcal{S}} \rightarrow \mathbb{R}^{\#\mathcal{S}}$$, called *backup operator*

    $$
    V_{n+1} = [\mathcal{T} V_n](s) = \max_a \mathbb{E}_{s' \vert s, a} [ r + \gamma V_n(s') ]
    $$

-   $$\mathcal{T}$$ is a contraction, that is

    $$
    V, \mathcal{T} V, \mathcal{T}^2 V, \cdots \rightarrow V^{*, \gamma}
    $$

#### Policy Iteration

**Problem:** evaluate fixed policy $$\pi$$

$$
V^{\pi, \gamma}(s) = \mathbb{E} [ r_0 + \gamma r_1 + \gamma^2 r_2 + \cdots \vert s_0 = s ]
$$

Backwards recursion involves a backup operator $$V_t = \mathcal{T}^\pi V_{t+1}$$ where

$$
[\mathcal{T}^\pi V] = \mathbb{E}_{s' \vert s, a=\pi(s)} [ r + \gamma V(s') ]
$$

Alternate between  
1.  Evaluate policy $$\pi \Rightarrow V^\pi$$
2.  Set new policy to be *greedy* policy for $$V^\pi$$

    $$
    \pi(s) = \arg \max_a \mathbb{E}_{s' \vert s, a} [ r + \gamma V^\pi(s') ]
    $$

#### Modified Policy Iteration

Update $$\pi$$ to be the greedy policy, then value function with $$k$$ backups ($$k$$-step lookahead)

$$
\begin{align}
&\text{Initialize } V^{(0)} \text{.} \\
&\textbf{for } n = 1, 2, \dots \textbf{ do} \\
&\hspace{1em} \pi^{(n+1)} = \mathcal{G} V^{(n)} \\
&\hspace{1em} V^{(n+1)} = (\mathcal{T}^{\pi^{(n+1)}}) V^{(n)} , \text{ for integer } k \geq 1 \\
&\textbf{end for}
\end{align}
$$

-   $$k=1$$: value iteration
-   $$k=\infty$$: policy iteration

---

## Policy Gradient Methods

### Parameterized Policies

Analogous to classification or regression with input $$s$$, output $$a$$.  
-   Discrete action space (classification): network outputs vector of probabilities
-   Continuous action space (regression): network outputs mean and diagonal covariance of Gaussian

### Prolicy Gradient Methods: Overview

**Goal**  

$$
\text{maximize} \, \mathbb{E} [ R \vert \pi_\theta ]
$$

**Intuitions**  

Collect a bunch of trajectories, and...  
-   Make the good trajectories more probable
-   Make the good actions more probable
-   Push the actions towards good actions

#### Score Function Gradient Estimator

Have expectation $$\mathbb{E}_{x \sim p(x \vert \theta)} [ f(x) ]$$, want gradient w.r.t. $$\theta$$.  

$$
\begin{align}
\nabla_\theta \mathbb{E}_x [ f(x) ]
&= \nabla_\theta \int dx \, p(x \vert \theta) f(x) \\
&= \int dx \, \nabla_\theta p(x \vert \theta) f(x) \\
&= \int dx \, p(x \vert \theta) \frac{\nabla_\theta p(x \vert \theta)}{p(x \vert \theta)} f(x) \\
&= \int dx \, p(x \vert \theta) \nabla_\theta \log{p(x \vert \theta)} f(x) \\
&= \mathbb{E}_x [ f(x) \nabla_\theta \log{p(x \vert \theta)} ] \,.
\end{align}
$$

>   **Hack**  
>   Switch place for $$\nabla_\theta$$ and $$\int dx$$.  

Now we have the gradient  

$$
\hat{g}_i = f(x_i) \nabla_\theta \log{p(x \vert \theta)}
$$

Then substitute $$f(x)$$ with whatever you want, say $$R(\tau)$$, where $$x$$ is substituted with the whole trajectory $$\tau = (s_0, a_0, r_0, s_1, a_1, r_1, \dots, s_{T-1}, a_{T-1}, r_{T-1}, s_T)$$, we get  

$$
\nabla_\theta \mathbb{E}_\tau [ R(\tau) ] = \mathbb{E}_\tau [ \nabla_\theta \log{p(\tau \vert \theta)} R(\tau) ]
$$

Then we look at $$p(\tau \vert \theta)$$.  

$$
\begin{align}
p(\tau \vert \theta) &= \mu(s_0) \prod_{t=0}^{T-1} [ \pi(a_t \vert s_t, \theta) P(s_{t+1, r_t \vert s_t, a_t}) ] \\
\log{p(\tau \vert \theta)} &= \log{\mu(s_0)} + \sum_{t=0}^{T-1} [ \log{\pi(a_t \vert s_t, \theta)} + \log{P(s_{t+1}, r_t \vert s_t, a_t)} ] \\
\nabla_\theta \log{p(\tau \vert \theta)} &= \nabla_\theta \sum_{t=0}^{T-1} \log{\pi(a_t \vert s_t, \theta)} \\
\nabla_\theta \mathbb{E}_\tau[R] &= \mathbb{E}_\tau \bigg[ R \, \nabla_\theta \sum_{t=0}^{T-1} \log{\pi(a_t \vert s_t, \theta)} \bigg]
\end{align}
$$

#### Introduce Baseline

Further reduce variance by introducing baseline $$b(S)$$.  

$$
\nabla_\theta \mathbb{E}_\tau[R] = \mathbb{E}_\tau \bigg[ \sum_{t=0}^{T-1} \nabla_\theta \log{\pi(a_t \vert s_t, \theta)} \bigg( \sum_{t' = t}^{T-1} r_{t'} - b(s_t) \bigg) \bigg]
$$

-   Near optimal choice is expected return,

    $$
    b(s_t) \approx \mathbb{E} [ r_t + r_{t+1} + \cdots + r_{T-1} ]
    $$

**Discounts for Variance Reduction**  

Introduce discount factor $$\gamma$$, which ignores delayed effects between actions and rewards.  

$$
\nabla_\theta \mathbb{E}_\tau[R] \approx \mathbb{E}_\tau \bigg[ \sum_{t=0}^{T-1} \nabla_\theta \log{\pi(a_t \vert s_t, \theta)} \bigg( \sum_{t' = t}^{T-1} \gamma^{t' - t} r_{t'} - b(s_t) \bigg) \bigg]
$$

-   want

    $$
    b(s_t) \approx \mathbb{E} [ r_t + \gamma r_{t+1} + \gamma^2 r_{t+2} + \cdots + \gamma^{T-1-t} r_{T-1} ]
    $$

### "Vanilla" Policy Gradient Algorithm

$$
\begin{align}
&\text{Initialize policy parameter } \theta, \text{ baseline } b \\
&\textbf{for} \text{ iteration} = 1, 2, \dots \ \textbf{do} \\
&\hspace{1em} \text{Collect a set of trajectories by executing current policy} \\
&\hspace{1em} \text{At each timestep in each trajectory, compute} \\
&\hspace{2em} \text{the} \textit{ return } R_t = \sum_{t'=t}^{T-1} \gamma^{t'-t} r_{t'} \text{, and} \\
&\hspace{2em} \text{the} \textit{ advantage estimate } \hat{A}_t = R_t - b(s_t) \text{.} \\
&\hspace{1em} \text{Re-fit the baseline, by minimizing } \| b(s_t) - R_t \| ^2 \\
&\hspace{2em} \text{summed over all trajectories and timesteps.} \\
&\hspace{1em} \text{Update the policy, using a policy gradient estimate } \hat{g} \text{,} \\
&\hspace{2em} \text{which is a sum of terms } \nabla_\theta \log{\pi(a_t \vert s_t, \theta)} \hat{A}_t \text{.} \\
&\textbf{end for}
\end{align}
$$

### Value Functions

-   Q-Function / state-action-value function

    $$
    Q^{\pi, \gamma}(s, a) = \mathbb{E}_\pi [ r_0 + \gamma r_1 + \gamma^2 r_2 + \cdots \vert s_0 = s, a_0 = a ]
    $$

-   state-value function

    $$
    \begin{align}
    V^{\pi, \gamma}(s)
    &= \mathbb{E}_\pi [ r_0 + \gamma r_1 + \gamma^2 r_2 + \cdots \vert s_0 = s ] \\
    &= \mathbb{E}_{a \sim \pi} [ Q^{\pi, \gamma}(s, a) ]
    \end{align}
    $$

-   advantage function

    $$
    A^{\pi, \gamma}(s, a) = Q^{\pi, \gamma}(s, a) - V^{\pi, \gamma}(s)
    $$

### Policy Gradient Formulas with Value Functions

$$
\begin{align}
\nabla_\theta \mathbb{E}_\tau [ R ]
&= \mathbb{E}_\tau \bigg[ \sum_{t=0}^{T-1} \nabla_\theta \log{\pi(a_t \vert s_t, \theta)} Q^{\pi}(s_t, a_t) \bigg] \\
&= \mathbb{E}_\tau \bigg[ \sum_{t=0}^{T-1} \nabla_\theta \log{\pi(a_t \vert s_t, \theta)} A^\pi(s_t, a_t) \bigg] \\
&\approx \mathbb{E}_\tau \bigg[ \sum_{t=0}^{T-1} \nabla_\theta \log{\pi(a_t \vert s_t, \theta)|} A^{\pi, \gamma}(s_t, a_t) \bigg]
\end{align}
$$



---

# Assignments

Code could be found on [GitHub](https://github.com/smdsbz/CS294-assignment).  

## Assignment 1

**Section 1: Getting Set Up**  

-   Out-Dated Pre-Trained `.pkl`
    -   Bit of hack: replace all `v1` seen, could be found in `expert/` and `demo.bash`, with `v2`.  
-   `mujoco-py` Installation Related *(tested on Ubuntu 18.04)*
    -   `GL/osmesa.h` not found
        -   `sudo apt install libosmesa6-dev`
    -   `-lGL` not found from `ld`
        -   `sudo apt install libgl*` (`*` is the *nix wildcard, expand it yourself)
    -   `patchelf` command not found
        -   `sudo apt install pathelf`

**Section 2: Warmup**  

1.  dump the `expert_data` in `hw1/run_expert.py` to `pkl`, will be used as training data
2.  define network
    -   model: simple NN
    -   loss: MSE
    -   optimizer: SGD
3.  train

For code, see [GitHub Repo](https://github.com/smdsbz/CS294-assignment/blob/master/hw1/WarmUp.ipynb).  

---

## Assignment 2

### Section 4: Implement Policy Gradient

**Networks**  

1. Use `build_mlp` (Section 3) to create policy net, which takes in `observations` and gives logits of `actions`
2. Choose / Sample one from `actions` based on calculated logits
3. Calculate log probability of `actions` aginst their labels, i.e. error

- `tf.nn.softmax_cross_entropy_with_logits_v2`
    - Measures the probability error in **distance** classification tasks in which the classes are mutually exclusive.
    - `logits` and `labels` must have the same shape, e.g. `[batch_size, num_classes]` and the same dtype (either `float16`, `float32` or `float64`)
    - Backpropagation will happen into both `logits` and `labels`. To disallow backpropagation into `labels`, pass label tensors through `@{tf.stop_gradient}` before feeding it to this function
- `tf.nn.sparse_softmax_cross_entropy_with_logits`
    - Measures the probability error in **discrete** classification tasks in which the classes are mutually exclusive.
    - A common use case is to have logits of shape `[batch_size, num_classes]` and labels of shape `[batch_size]`.

> **Note**  
> View **error** from a stocastic perspective: the higher an error is, the more the mean is deviated from the **most probable location** in the space (if you set label as **MPL** or **mean** roughly), i.e. less probability (or larger negative number in log space) at the spot where the error is calculated.  
> In short, $$- \log{(\text{probability})} \Rightarrow \text{error}$$.  

**Computing Q-Values**  

- Policy gradient: $$\hat{g} = \mathbb{E}_\tau [ \sum_{t=0}^{T} \nabla \log{\pi(a_t \vert s_t)} \cdot (Q_t - b_t) ]$$
- Q-value: $$Q_t$$
    - trajectory-based (all along trajectory): $$\sum_{t'=0}^{T} \gamma^{t'} r_{t'}$$
    - reward-to-go (from current timestep to future): $$\sum_{t'=t}^{T} \gamma^{t'-t} r_{t'}$$

For code, see [GitHub Repo](https://github.com/smdsbz/CS294-assignment/blob/master/hw2/train_pg.py)

