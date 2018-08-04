---
layout: article
title: CS294 Deep Reinforcement Learning Notes
key: cs294-drl-learning-notes
tags: MachineLearning DeepLearning ReinforcementLearning
---

<!-- more -->

{:toc}

---

# Lectures

## The "REPL"

### The Roles

- Given by **Agent**

    $$
    \pi_\theta(\mathbf{u}_t \vert \mathbf{o}_t) \,, \mathbf{o}_t \rightarrow \mathbf{x}_t
    $$

    - $$\pi$$ as *policy*, $$\theta$$ as parameters in policy model

        >   **Note**  
        >   Action space could be discrete, consider using **Monte-Carlo Tree Search** then.  

    - $$\mathbf{u}$$, $$\mathbf{x}$$, $$\mathbf{o}$$ as *action*, *ground-truth state* and *observed state* respectively, $$t$$ for current time step

- Given by **Environment**

    $$
    \mathrm{p}(\mathbf{x}_{t+1} \vert \mathbf{x}_t, \mathbf{u}_t)
    $$

    - $$\mathrm{p}$$ (stocastically) or $$f$$ for set of state transition rules built within the environment

- Defined by **User**

    $$
    r(\mathbf{x}, \mathbf{u})
    $$

    - $$r$$ for *reward*, or $$c := -r$$ for *cost*

> **Note**  
> Clever audiences may have found the **3 Places to Apply NN** in a RL model by now.  
> - **Agent:** learn Policies
> - **Environment:** learn Dynamics
> - **Feedback:** learn Rewards


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

> **Note**  
> $$\mathbf{x}$$ may contain quadraic terms of its elements, velocity $$\dot{x}$$ and acceleration $$\ddot{x}$$ for example.  

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

> **Flags**  
> - [x] You'd want to touch this thing no more
> - [ ] It can be solved properly and easily with an NN (and escape from Newton's Method for good)

### Nonlinear Case

Approximate linear-quadratic case using Taylor expansion.  

- Differential dynamic programming (or DDP)
- Iterative LQR
    - For all reference points, run LQR backward recursion on approximation, then run forward recursion on true nonlinear system (with hacks that will not be discussed on this post)

---

## Learn the Model / Dynamics

**Goal**  

Discover $$p(\mathbf{x}_{t+1} \vert \mathbf{x}_t, \mathbf{u}_t)$$.  

### Model-Based RL v1.5

1.  Explore the playground with a particular goal
    - Run base policy $$\pi_0(\mathbf{u}_t \vert \mathbf{x}_t)$$ to collect $$\mathcal{D}=\{ (\mathbf{x}, \mathbf{u}, \mathbf{x}')_i \}$$
2.  Learn from exprerience to generate $$p_{\pi_0}$$, the observed model
    - Learn dynamics model $$f(\mathbf{x}, \mathbf{u})$$ to minimize $$\sum_i \lVert f(\mathbf{x}_i, \mathbf{u}_i) - \mathbf{x}_i' \rVert ^2$$
3.  Dream of more adventures
    - Backpropagate through $$f(\mathbf{x}, \mathbf{u})$$ to choose actions
4.  Take real actions, **verify the learnt exprerience**, i.e. test observed model $$p_{\pi_0}$$ against ground-truth $$p_{\pi_f}$$ to find if any mismatch exists, and try to get even close to our goal
    - [v1.5] (MPC) Execute those actions and observe consequent states $$\mathbf{x}'$$
    - (DAgger) append the resulting data $$\{ (\mathbf{x}, \mathbf{u}, \mathbf{x}')_j \}$$ to $$\mathcal{D}$$
    - [v1.5] Back to step 3 to **re-plan** actions (correct errors before going off road, planning algorithm required), loop for N times (or N-parallelism)
    - back to step 2

### Model-Based RL v2.0

1.  Explore the playground with a particular goal
    - Run base policy $$\pi_0(\mathbf{u}_t \vert \mathbf{x}_t)$$ to collect $$\mathcal{D}=\{ (\mathbf{x}, \mathbf{u}, \mathbf{x}')_i \}$$
2.  Learn from exprerience to generate $$p_{\pi_0}$$, the observed model
    - Learn dynamics model $$f(\mathbf{x}, \mathbf{u})$$ to minimize $$\sum_i \lVert f(\mathbf{x}_i, \mathbf{u}_i) - \mathbf{x}_i' \rVert ^2$$
3.  Dream of more adventures
    - Backpropagate through $$f(\mathbf{x}, \mathbf{u})$$ into the policy (along the **whole** decision chain) to optimze $$\pi_\theta(\mathbf{u}_t \vert \mathbf{x}_t)$$
    - Run $$\pi_\theta(\mathbf{u}_t \vert \mathbf{x}_t)$$
4.  Take real actions, and try to get even close to our goal
    - (DAgger) append the resulting data $$\{ (\mathbf{x}, \mathbf{u}, \mathbf{x}')_j \}$$ to $$\mathcal{D}$$
    - Back to step 2

---

## Train Policies (Model-Based RL)

- No need to replan
- Potentially better generalization

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
    - Find a good trajectory
    - Optimize $$p(\tau)$$ w.r.t. some surrogate $$\tilde{c}(\mathbf{x}_t, \mathbf{u}_t)$$
    - Sequential, may use iLQR
2.  Find $$\theta \leftarrow \arg \min_\theta \bar{\mathcal{L}}(\tau, \theta, \lambda)$$
    - learn (imitational) the policy upon the trajectory
    - Optimize $$\theta$$ w.r.t. some supervised objective
    - non-sequential, may use SGD
3.  $$\lambda \leftarrow \lambda + \alpha \frac{dg}{d\lambda}$$
    - Do gradient descent
    - Increment or modify dual variables $$\lambda$$

### Imitating MPC: PLATO Algorithm

1.  Train $$\pi_\theta(\mathbf{u}_t \vert \mathbf{o}_t)$$ from human data $$\mathcal{D} = \{ \mathbf{o}_1, \mathbf{u}_1, \dots, \mathbf{o}_N, \mathbf{u}_N \}$$
2.  Run (in real dynamics, e.g. real world) $$\hat{\pi}(\mathbf{u}_t \vert \mathbf{o}_t)$$ to get dataset $$\mathcal{D} = \{ \mathbf{o}_1, \dots, \mathbf{o}_M \}$$
    - $$\hat{\pi}(\mathbf{u}_t \vert \mathbf{x}_t) = \arg \min_\hat{\pi} \sum_{t' = t}^T E_\hat{\pi} [ c(\mathbf{x}_{t'}, \mathbf{u}_{t'}) ] + \lambda D_{\mathrm{KL}}(\hat{\pi}(\mathbf{u}_t \vert \mathbf{x}_t) \Vert \pi_\theta(\mathbf{u}_t \vert \mathbf{x}_t))$$
        - Keep newly generated training trajectories different from but relatively **close** to original ones (could be human interfering manually via MPC), in order to avoid financially unacceptabe training cost, for it may lead to physical damage on the test object / environment (e.g. autopilot drone bump into a tree, the model will be taught a good lesson, but the drone is broken for good)
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

- Optimize over controls
- State trajectory is implicit
- Dynamics is an implicit constraint (always satisfied)

**Direct Collocation**  

$$
\min_{\mathbf{x}_1, \dots, \mathbf{x}_T} \sum_t c_t(\mathbf{x}_t) \text{ s.t. } f^{-1}(\mathbf{x}_t, \mathbf{x}_{t+1}) = \mathbf{u}_t \in \mathcal{U}
$$

- Optimize over states
- Controls and forces are implicit
- Dynamics is explict constraint (can be made hard or soft)

### Dynamics with Contact

**Problems**  

- Discontinuous jumps in contact forces (and their number)
- No gradient information from inactive contacts

**Contact-Invariant Optimization**  

- $$c_{t, n} = 1$$, the controller $$n$$ must be touching something and not sliding
- $$c_{t, n} = 0$$, the controller $$n$$ is unconstrained

**Dynamics Consistency**  

Assume all forces are active (contact set is constant), but apply high penalties for use of forces where $$c = 0$$.  

$$
f^{-1}(\mathbf{x}_{t-1}, \mathbf{x}_{t}, \mathbf{x}_{t+1}) = \arg \min_{\mathbf{u}, \mathbf{f}} \lVert \text{ difference of dynamics equation } \rVert ^2 + \sum_{i} \lVert \mathbf{f}_i \rVert ^2 / (c_i + \epsilon)
$$

### Policy Drifting

Supervised learning in RL is different from traditional supervised learning, like in recognition tasks, in the way that the *steps* are now no longer independent of time / sequence with each other. Errors in feed-forward results will accumulate, getting the model into states that are off the labeled trajectory.  

**Solution: Noisy Training Data**  

Simulate expert policies at states that are slightly off, by generating a correction term pointing towards the opposite direction of the error.  

- Input: $$\mathbf{x} + \epsilon$$
- Output: $$\mathbf{u} + \mathbf{K} \epsilon$$

### Decompose Policy Optimization Problem

- Trajectory optimization
    - Stay close to policy

    $$
    \min_{\mathbf{x}} \sum_t C(\mathbf{x}_t) + \lVert \pi_\theta(\mathbf{x}_t) - \mathbf{u}_t \rVert ^2
    $$

- Regression

  $$
  \min_\theta \sum_{i, t} \lVert \pi_\theta(\mathbf{x}_{i, t}) - \mathbf{u}_{i, t} \rVert ^2
  $$

### Movement with Model Uncertianty

- Generate noisy models varying:
    - limb mass
    - limb conter of mass
    - contact locations
    - etc.
- Optimize over multiple state trajectories
- Single control trajectory
- Execute control trajectory in open loop

### Interactive Policies with Online Model Learning

- Offline
    - Train policy to output desired next state $$\bar{\mathbf{x}}_{t+1}$$
- At every timestep
    - Learn robot dynamics on the fly from past observations $$\mathbf{x}_{t+1} = f(\mathbf{x}_t, \mathbf{u}_t)$$
    - Query policy for $$\bar{\mathbf{x}}_{t+1}$$
    - Solve for robot torques $$\mathbf{u}^*$$ such that $$\bar{\mathbf{x}}_{t+1} = f(\mathbf{x}_t, \mathbf{u}^*)$$

---

## Markov Decision Processes and Solving Finite Problems

### Markov Decision Process

Defined by the following components:  
- $$\mathcal{S}$$: **state space**
- $$\mathcal{A}$$: **action space**
- $$P(r, s' \vert s, a)$$: a transition probability distribution
    - $$r$$: reward
    - $$s$$, $$s'$$: state and next state

### Partially Observed MDPs

- Instead of obsrving full state $$s$$, agent observes $$y$$ (previously denoted as $$o$$), with $$y \sim P(y \vert s)$$
- A MDP can be trivially mapped onto a POMDP
- A POMBDP can be mapped on to an MDP

  $$
  \tilde{s}_0 = \{ y_0 \} ,\, \tilde{s}_1 = \{ y_0, y_1 \} ,\, \tilde{s}_2 = \{ y_0, y_1, y_2 \} ,\, \dots
  $$


### Problems Involving MDPs

- **Policy optimization**: maximize expected reward w.r.t. policy $$\pi$$

    $$
    \max_\pi \mathbb{E} \bigg[ \sum_{t=0}^\infty r_t \bigg]
    $$

- **Policy evaluation**: compute exprected return for fixed policy $$\pi$$
    - return := sum of future rewards in an episode, i.e. a trajectory
        - Discounted return: $$r_t + \gamma r_{t+1} + \gamma^2 r_{t+2} + \cdots$$
        - Undiscounted return: $$r_t + r_{t+1} + \cdots + r_{T-1} + V(s_T)$$
    - Performance of policy: $$ \eta(\pi) = \mathbb{E} [ \sum_{t=0}^\infty \gamma^t r_t ] $$
    - State value function: $$ V^\pi(s) = \mathbb{E} [ \sum_{t=0}^\infty \gamma^t r_t \vert s_0 = s ] $$
    - State-action value function: $$ Q^\pi(s, a) = \mathbb{E} [ \sum_{t=0}^\infty \gamma^t r_t \vert s_0 = s, a_0 = a ] $$

#### Policy Evaluation

##### Value Iteration: Finite Horizon Case

_General idea of LQR algorithm._  

- Problem

    $$
    \min_{\pi_0, \dots, \pi_{T-1}} \mathbb{E}[ r_0 + r_1 + \cdots + r_{T-1} + V_T(s_T) ]
    $$

- Swap maxes and exprectations

    $$
    \max_{\pi_0} \mathbb{E} \bigg[ r_0 + \max_{\pi_1} \mathbb{E} \big[ r_1 + \cdots + \max_{\pi_{T-1}} \mathbb{E} [ r_{T-1} + V_T(s_T) ] \big] \bigg]
    $$

- Solve innermost problem

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

- Discount factor $$\gamma \in [0, 1)$$, downweights future rewards
- Discounted return: $$r_0 + \gamma r_1 + \gamma^2 r_2 + \cdots$$
- Effective time horizon $$(1 + \gamma + \gamma^2 + \cdots) = 1 / (1 - \gamma)$$
- Discounted problem can be obtained by adding transitions to *sink state*, where agent gets stuck and recieves zero reward

    $$
    \tilde{P}(s' \vert s, a) = \begin{cases}
    P(s' \vert s, a) \text{ with probability } \gamma \\
    \text{sink state with probability } 1 - \gamma
    \end{cases}
    $$


##### Infinite-Horizon V.I. Via Finite-Horizon V.I.

- Pretend there exists finite horizon $$T$$, ignore $$r_T, r_{T+1}, \dots$$
    - Error $$\epsilon \leq r_\max \gamma^T / (1 - \gamma)$$
    - Resulting nonstationary policy only suboptimal by $$\epsilon$$
    - $$\pi_0, V_0$$ converges to optimal policy as $$T \rightarrow \infty$$

##### Infinite-Horizon V.I. Via Operator View

- $$V \in \mathbb{R}^{\#\mathcal{S}}$$: $$V$$ is a vector with number of dimensions equal to the degree of freedomness of state
- V.I. update is a function $$\mathcal{T}: \mathbb{R}^{\#\mathcal{S}} \rightarrow \mathbb{R}^{\#\mathcal{S}}$$, called *backup operator*

    $$
    V_{n+1} = [\mathcal{T} V_n](s) = \max_a \mathbb{E}_{s' \vert s, a} [ r + \gamma V_n(s') ]
    $$

- $$\mathcal{T}$$ is a contraction, that is

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
1. Evaluate policy $$\pi \Rightarrow V^\pi$$
2. Set new policy to be *greedy* policy for $$V^\pi$$

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

- $$k=1$$: value iteration
- $$k=\infty$$: policy iteration

---

## Policy Gradient Methods

### Parameterized Policies

Analogous to classification or regression with input $$s$$, output $$a$$.  
- Discrete action space (classification): network outputs vector of probabilities
- Continuous action space (regression): network outputs mean and diagonal covariance of Gaussian

### Prolicy Gradient Methods: Overview

**Goal**  

$$
\text{maximize} \, \mathbb{E} [ R \vert \pi_\theta ]
$$

**Intuitions**  

Collect a bunch of trajectories, and...  
- Make the good trajectories more probable
- Make the good actions more probable
- Push the actions towards good actions

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

- Near optimal choice is expected return,

    $$
    b(s_t) \approx \mathbb{E} [ r_t + r_{t+1} + \cdots + r_{T-1} ]
    $$


**Discounts for Variance Reduction**  

Introduce discount factor $$\gamma$$, which ignores delayed effects between actions and rewards.  

$$
\nabla_\theta \mathbb{E}_\tau[R] \approx \mathbb{E}_\tau \bigg[ \sum_{t=0}^{T-1} \nabla_\theta \log{\pi(a_t \vert s_t, \theta)} \bigg( \sum_{t' = t}^{T-1} \gamma^{t' - t} r_{t'} - b(s_t) \bigg) \bigg]
$$

- Want

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

- Q-Function / state-action-value function

    $$
    Q^{\pi, \gamma}(s, a) = \mathbb{E}_\pi [ r_0 + \gamma r_1 + \gamma^2 r_2 + \cdots \vert s_0 = s, a_0 = a ]
    $$

- State-value function

    $$
    \begin{align}
    V^{\pi, \gamma}(s)
    &= \mathbb{E}_\pi [ r_0 + \gamma r_1 + \gamma^2 r_2 + \cdots \vert s_0 = s ] \\
    &= \mathbb{E}_{a \sim \pi} [ Q^{\pi, \gamma}(s, a) ]
    \end{align}
    $$

- Advantage function

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

## Q-Function Learning Methods

_(model-free reinforcement learning method)_

### Bellman Equations for $Q^\pi$

$$
\begin{align}
Q^\pi(s_0, a_0)
&= \mathbb{E}_{s_1 \sim P(s_1 \vert s_0, a_0)} [ r_0 + \gamma V^\pi(s_1) ] \\
&= \mathbb{E}_{s_1 \sim P(s_1 \vert s_0, a_0)} [ r_0 + \gamma \mathbb{E}_{a_1 \sim \pi} [ Q^\pi(s_1, a_1) ] ]
\end{align}
$$

Define the Bellman backup operator (operating on Q functions) as follows  

$$
[\mathcal{T}^\pi Q](s_0, a_0) = \mathbb{E}_{s_1 \sim P(s_1 \vert s_0, a_0)} [ r_0 + \gamma \mathbb{E}_{a_1 \sim \pi}[Q(s_1, a_1)] ]
$$

### Introducing $Q^\star$

- Let $$\pi^\star$$ denote an optimal policy
- Define $$Q^\star = Q^{\pi^\star}$$, which satisfies $$Q^\star(s, a) = \max_\pi Q^{\pi}(s, a)$$
- $$\pi^\star$$ satisfies $$\pi^\star(s) = \arg \max_a Q^\star(s, a)$$

Therefore,  

$$
Q^\pi(s_0, a_0) = \mathbb{E}_{s_1 \sim P(s_1 \vert s_0, a_0)} [ r_0 + \gamma \max_{a_1} Q^\star(s_1, a_1) ]
$$

Get *(Q-Value iteration)*  

$$
\lim_{t \rightarrow +\infty} \mathcal{T}^t Q \rightarrow Q^\star
$$

### Why $Q$ Rather than $V$

- Can compute greedy aciton $$\max_a Q(s, a)$$ without knowing $$P$$
- Can compute unbiased estimator of backup value $$[\mathcal{T}Q](s, a)$$ without knowing $$P$$ using single transition $$(s, a, r, s')$$
- Can compute unbiased estimator of backup value $$[\mathcal{T}Q](s, a)$$ using off-policy data

*(potentially model-free)*  

### Partial Backups

*(reduce noise)*  

- Partial backup: $$Q \leftarrow \epsilon \widehat{\mathcal{T}Q} + (1 + \epsilon) Q$$
- Equivalent to gradient step on squared error

    $$
    \begin{align}
    Q
    &\rightarrow Q - \epsilon \nabla_Q \lVert Q - \widehat{\mathcal{T} Q_t} \rVert ^2 / 2 \\
    &= Q - \epsilon (Q - \widehat{\mathcal{T} Q_t}) \\
    &= (1 - \epsilon) Q + \epsilon \widehat{\mathcal{T} Q_t}
    \end{align}
    $$


### Sampling-Based Q-Value Iteration

$$
\begin{align}
&\text{Initialize } Q^{(0)} \\
&\textbf{for } n = 0, 1, 2, \dots \text{ until termination conditon } \textbf{do} \\
&\hspace{1em} \text{Interact with the environment for } K \text{ timesteps (including multiple episodes)} \\
&\hspace{1em} Q^{(n+1)} = Q^{(n)} + \epsilon \nabla_Q \sum_{t=1}^{K} \lVert \widehat{\mathcal{T} Q_t} - Q(s_t, a_t) \rVert ^2 / 2 \\
&\textbf{end for}
\end{align}
$$

> **Note**  
> Appropriate schedule required for convergence, e.g. $$\epsilon = 1 /n$$.  

### Function Approximation / Neural-Fitted Algorithms

- Parameterize Q function with a neural network $$Q_\theta$$
- To approximate $$Q \leftarrow \widehat{\mathcal{T}Q}$$, do

    $$
    \underset{\theta}{\text{minimize}} \sum_t \lVert Q_\theta(s_t, a_t) - \widehat{\mathcal{T}Q}(s_t, a_t) \rVert ^2
    $$


---

## Deep Q-Network

### Q-Value Iteration with Function Approximation

$$
\begin{align}
&\text{Initialize } \theta^{(0)} \\
&\textbf{for } n = 0, 1, 2, \dots \textbf{do} \\
&\hspace{1em} \text{Run policy for } K \text{ timesteps using some policy } \pi^{(n)} \\
&\hspace{1em} g^{(n)} = \nabla_\theta \sum_{t} \bigg( \widehat{\mathcal{T} Q_t} - Q_\theta(s_t, a_t) \bigg) ^2 \\
&\hspace{1em} \theta^{(n+1)} = \theta^{(n)} - \alpha g^{(n)} \\
&\textbf{end for}
\end{align}
$$

### Key Terms

- Replay memory $$\mathcal{D}$$: history of last $$N$$ transitions
- Target network: old Q function $$Q^{(n)}$$ that is fixed over many (~ 10,000) timesteps, while $$Q \Rightarrow \mathcal{T}Q^{(n)}$$

### Deep Q-learning with Experience Replay

$$
\begin{align*}
&\text{Initialize replay memory } \mathcal{D} \text{ to capacity } N \\
&\text{Initialize action-value function } Q \text{ with random weights} \\
&\textbf{for} \text{ episode } = 1, \cdots, M \textbf{ do} \\
    &\hspace{1em} \text{Initialize sequence } s_1 = \{ x_1 \} \text{ and preprocessed sequenced } \phi_1 = \phi(s_1) \\
    &\hspace{1em} \textbf{for } t = 1, \cdots, T \textbf{ do} \\
        &\hspace{2em} \text{With probability } \epsilon \text{ select a random action } a_t \\
            &\hspace{4em} \text{otherwise select } a_t = \max_a Q^*(\phi(s_t), a; \theta) \\
        &\hspace{2em} \text{Set } s_{t+1} = s_t, a_t, x_{t+1} \text{ and preprocess } \phi_{t+1} = \phi(s_{s+1}) \\
        &\hspace{2em} \text{Store transition } (\phi_t, a_t, r_t, \phi_{t+1}) \text{ in } \mathcal{D} \\
        &\hspace{2em} \text{Sample random minibatch of transitions } (\phi_j, a_j, r_j, \phi_{j+1}) \text{ from } \mathcal{D} \\
        &\hspace{2em} \text{Set } y_j = \begin{cases}
                                        r_j                                              &\text{for terminal } \phi_{j+1} \\
                                        r_j + \gamma \max_{a'} Q(\phi_{j+1}, a'; \theta) &\text{for non-terminal } \phi_{j+1} \\
                                        \end{cases} \\
        &\hspace{2em} \text{Perform a gradient descent step on } (y_j - Q(\phi_j, a_j; \theta))^2 \\
    &\hspace{1em} \textbf{end for} \\
&\textbf{end for} \\
\end{align*}
$$

- $$\phi$$: preprocessed input, e.g. image, $$\approx s$$
- $$\gamma$$: our good ol' discount factor
    - Assuming actions far in the past contribute less to the present, for their influences are *indirect*

### Double Q-learning

- $$\mathbb{E}_{x_1, x_2}[\max(x_1, x_2)] \geq \max (\mathbb{E}_{x_1, x_2}[x_1], \mathbb{E}[x_2])$$
- Q values are noisy, thus $$r + \gamma \max_{a'} Q(s', a')$$ is an overestimate

**Solution**  

Use two networks $$Q_A, Q_B$$, and compute $$\arg\max$$ with the other network

$$
\begin{align}
Q_A(s, a) &\leftarrow r + \gamma Q(s', \arg\max_{a'} Q_B(s', a')) \\
Q_B(s, a) &\leftarrow r + \gamma Q(s', \arg\max_{a'} Q_A(s', a'))
\end{align}
$$

- Two Q functions of different noise, avoid overestimation and converges faster

### Dueling Net

$$
Q(s, a) = V(s) + A(s, a)
$$

$$\vert V \vert$$ has larger scale than $$\vert A \vert$$ by $$\approx 1 / (1 - \gamma)$$, for good states contributes more to the final goal than those good actions at given states do. But small differences $$A(s, a) - A(s, a')$$ determine policy, therefore they could be easily *submerged* by the swing in $$\vert V \vert$$!

**Solution**  

Parameterize Q function as follows  

$$
Q_\theta(s, a) = V_\theta(s) + \underbrace{F_\theta(s, a) - \text{mean}_{a'}F_\theta(s, a')}_{``\text{Advantage''} \text{ part}}
$$

### Prioritized Replay

- Bellman error loss: $$\sum_{i \in \mathcal{D}} \lVert Q_\theta(s_i, a_i) - \hat{Q}_t \rVert ^2 / 2$$
- Can use importance sampling to favor timestep $$i$$ with large gradient, allowing faster backwards propagation of reward info
- Use last Bellman error $$\vert \delta_i \vert$$, where $$\delta_i = Q_\theta(s_i, a_i) - \hat{Q}_t$$ as proxy for size of gradient

### Practical Tips

- User *Huber loss* on Bellman error

    $$
    \begin{align*}
    L(x) = \begin{cases}
           x^2 / 2 &\text{if } \vert x \vert \leq \delta \\
           \delta \vert x \vert - \delta^2 / 2 &\text{otherwise}
           \end{cases}
    \end{align*}
    $$

- Use Double DQN
- Try your own skills at navigating the environment based on processed frames, in order to test out your data preprocessing (e.g. image down-sampling)
- Always run at least two different seeds when experimenting
- Learning rate schedueling, try high ones in initial exploring period
- Try non-standard exploring schedules


---

## Artistic Stylization and Rendering

### Methods

- Procedural Methods
    - Pure mathematics formulas, trying to immitate what real artists are thinking.  
        - Pros
            - lovely results
            - very controllable
        - Cons
            - hard to design styles
            - complex to implement

- Patch-Based Methods
    - Find in source for similar patches / patterns.  

- Neural Methods
    - Pros: good results
    - Cons: no degree of control

### Color Channel

- Luminance channel
    - texture and line info
- Color channel
    - color info

> **Note**  
> Luminance channel is much more important! Consider image compressing with `jpeg`.  

### Adding Control to Neural Methods

- Color control with Luminance style transfer
- Space control: masking out areas to apply algorithm

## Geometry Understanding

### Data Labeling Approches "without Human-Hour"

- Flickr image tags
    - noisy
    - try to model cofidence for tags
- Online 3D models
    - built with hierarchy in mind
    - possible to use name of parts that come with the downloaded model
        1. embedding geometries
        2. clustering feature vectors
        3. labeling each cluster with name and position in hierarchy


---

## Advanced Topics in Imitation Learning & Safety

### Approaches to Provide Demonstrations for an Robotic Arm and Humanoid

- kinesthetic teaching
- teleoperation

### Uncertainty-Aware Collision Cost

$$
c_\text{collision}(\tau) \propto \text{speed} \cdot \big( \mathbb{E}[p(c_{t+H} \vert \tau)] + \sqrt{\mathrm{Var}[p(c_{t+H} \vert \tau)]} \big)
$$


---

## Inverse Reinforcement Learning

### Motivation & Definition

In the real world, human **dont't** get a score! The reward function is unclear.  

Previously, solved by introducing *behavioral cloning*, but...  
- no reasoning about outcomes or dynamics
- the expert might have different degrees of freedom

#### Inverse Optimal Control / Inverse Reinforcement Learning

Infer cost / reward function from expert demonstrations.  

Given...  
- state & action space
- roll-outs from $$\pi^*$$
- [optional] dynamics model

Goal...  
- recover reward function
- then use reward to get policy

**Problem**  

- Underdefined problem
- Difficult to evaluate a learned cost
- Demonstrations may not be precisely optimal

### Maximum Entropy Inverse RL

Want to have  
- high scores for good trajectories
- low scores for bad trajectories

Construct  

$$
p(\tau) \propto e^{-c(\tau)}
$$  

Goal becomes  

$$
\min_\pi \mathbb{E}_\pi[c(\tau)] - \mathrm{H}(\pi)
$$

Now consider how to minimize the cost  
- cross entropy

$$
\begin{align}
\theta
&= \arg \max_\theta \log{\prod_{\tau_d \in \mathcal{D}} p(\tau_d)} \\
&= \arg \min_\theta - \frac{1}{M} \sum_{\tau_d \in \mathcal{D}} \log{\frac{1}{z} e^{-c(\tau_d)}} \\
&= \arg \min_\theta \frac{1}{M} \sum_{\tau_d} c(\tau_d) + \log{\sum_\tau e^{-c(\tau)}} \\
\end{align}
$$

where  
- $$z := \sum_\tau e^{-c(\tau)}$$
- $$M$$ is amount of data in demonstration dataset $$\mathcal{D}$$

Consider using gradient descent method, define $$\mathcal{L}$$ by

$$
\mathcal{L} := \frac{1}{M} \sum_{\tau_d} c(\tau_d) + \log{\sum_\tau e^{-c(\tau)}}
$$

Therefore  

$$
\nabla_\theta \mathcal{L} = \frac{1}{M} \sum_{\tau_d} \frac{d c(\tau_d)}{d \theta} + \frac{1}{\sum_\tau e^{-c(\tau)}} \sum_\tau \big( e^{-c(\tau)} \cdot (-1) \cdot \frac{d c(\tau)}{d \theta} \big)
$$

Note that  

$$
p(\tau) = \frac{e^{-c(\tau)}}{\sum_\tau e^{-c(\tau)}}
$$

Have  

$$
\nabla_\theta \mathcal{L} = \frac{1}{M} \sum_{\tau_d} \frac{d c(\tau_d)}{d \theta} - \sum_\tau p(\tau \vert \theta, T) \frac{d c(\tau)}{d \theta}
$$

where  
- $$T$$ is transition dynamics

Substitute trajectories with their states  

$$
\nabla_\theta \mathcal{L} = \frac{1}{M} \sum_{\tau_d} \frac{d c(\tau_d)}{d \theta} - \sum_s p(s \vert \theta, T) \frac{d c(s)}{d \theta}
$$

Note  

$$
\begin{align}
p(\tau)
&= p(s_1) \prod_t \pi_\theta(a_t \vert s_t) T(s_{t+1} \vert s_t, a_t) \\
&= p(s_1) \prod_t p(a_t \vert s_t, \theta) p(s_{t+1} \vert s_t, a_t)
\end{align}
$$

To compute $$p(s \vert \theta, T)$$  
1. $$\pi(a \vert s)$$ w.r.t. $$c_\theta$$
2. do

    $$
    \begin{align}
    &\mu_1(s) = p(s_1 = s) \\
    &\textbf{for } t = 1 : T - 1 \textbf{ do} \\
    &\hspace{1em} \mu_{t+1}(s) = \sum_{a', s'} \mu_t(s') \pi(a' \vert s') p(s \vert s', a') \\
    &\textbf{end for}
    \end{align}
    $$

    where
    - $$\mu_t(s)$$ is probability visiting $$s$$ at $$t$$
    - $$a', s'$$ are action / state of previous timestep respectively

3. $$p(s \vert \theta, T) = \sum_t \mu_t(s)$$

### Scaling Inverse RL to Deep Cost Functions

Backpropagation :smile:  

### Inverse RL with Unkown Dynamics

Consider using importance sampling.  

$$
\begin{align}
z
&= \sum_\tau e^{-c(\tau)} \\
&\approx \sum_{\tau_s \sim q} \frac{e^{-c(\tau_s)}}{q(\tau_s)}
\end{align}
$$

For importance sampling, want large probability of sampling for dramatic costs.  

$$
\text{minimize} \ \mathrm{Var} \, [ q(\tau) \propto \vert e^{-c(\tau)} \vert ]
$$

Problem is the target $$q$$ is hard to guess. Instead, try to construct $$q$$ dynamically.  

$$
q = \arg \min_q \mathbb{E}_q[c(\tau)] - \mathrm{H}(q)
$$

Also, construct multiple $$q$$s to avoid introducing high variance to sample data.  

$$
v(\tau) = \frac{1}{K} \sum_k q_k(\tau)
$$


---

## Advanced Policy Gradient Methods: Natural Gradient, TRPO, and More

$$
\text{maximize}_\theta \, \sum_{n=1}^N \frac{\pi_\theta(a_n \vert s_n)}{\pi_{\theta_\text{old}}(a_n \vert s_n)} \hat{A}_n - C \cdot \overline{\mathrm{KL}}_{\pi_{\theta_\text{old}}}(\pi_\theta)
$$

### Truncated Natural Policy Gradient Algorithm

- Unconstrained problem: $$\text{maximize} \, L_{\pi_{\theta_\text{old}}}(\pi_\theta) - C \cdot \overline{\mathrm{KL}}_{\pi_{\theta_\text{old}}}(\pi_\theta)$$

$$
\begin{align}
&\textbf{for} \text{ iteration} = 1, 2, \dots \textbf{ do} \\
&\hspace{1em} \text{Run policy for } T \text{ timesteps or } N \text{ trajectories} \\
&\hspace{1em} \text{Estimate advantage function at all timesteps} \\
&\hspace{1em} \text{Compute policy gradient } g \\
&\hspace{1em} \text{Use } \textit{Conjugated Gradient Method} \text{ (with Hessian-vector products) to compute } H^{-1}g \\
&\hspace{1em} \text{Update policy parameter } \theta = \theta_\text{old} + \alpha H^{-1}g \\
&\textbf{end for}
\end{align}
$$

### TRPO

- Constrained problem: $$\text{maximize} \, L_{\pi_{\theta_\text{old}}}(\pi_\theta)$$ subject to $$\overline{\mathrm{KL}}_{\pi_{\theta_\text{old}}}(\pi_\theta) \leq \delta$$
    - Set hyperparameter $$\delta$$ rather than $$C$$

$$
\begin{align}
&\textbf{for} \text{ iteration} = 1, 2, \dots \textbf{ do} \\
&\hspace{1em} \text{Run policy for } T \text{ timesteps or } N \text{ trajectories} \\
&\hspace{1em} \text{Estimate advantage function at all timesteps} \\
&\hspace{1em} \text{Compute policy gradient } g \\
&\hspace{1em} \text{Use } \textit{Conjugated Gradient Method} \text{ (with Hessian-vector products) to compute } H^{-1}g \\
&\hspace{1em} \text{Compute rescaled step } s = \alpha H^-1 g \text{ with rescaling and line search} \\
&\hspace{1em} \text{Apply update: } \theta = \theta_\text{old} + \alpha H^{-1}g \\
&\textbf{end for}
\end{align}
$$

### Alternative Method for Calculating Natural Gradients

- Fisher information matrix

    $$
    \frac{\partial}{\partial^2 \theta} \mathrm{KL}[p_{\theta_\text{old}}, p_\theta] = \mathbb{E}_{x \sim p_{\theta_\text{old}}} \bigg[ \bigg( \frac{\partial}{\partial \theta} \log{p_\theta(x)} \bigg)^T \bigg( \frac{\partial}{\partial \theta} \log{p_\theta(x)} \bigg) \bigg] \vert_{\theta = \theta_\text{old}}
    $$

- In policy optimization setting, instead of forming FIM by differentiating $$\mathrm{KL}$$, can explicitly form $$\sum_n \big( \frac{\partial}{\partial \theta} \log{\pi_\theta(a_n \vert s_n)} \big) ^T \big( \frac{\partial}{\partial \theta} \log{\pi_\theta(a_n \vert s_n)} \big)$$


---

## Variance Reduction for Policy Gradient Methods

### Reward Shaping

Giving a hint to the model, via a shaping term in the reward / cost function.  

$$\tilde{r}(s, a, s') = r(s, a, s') + \gamma \phi(s') - \phi(s)$$ for arbitrary *potential function* $$\phi$$.  

- Reward shaping transformation leaves policy gradient and optimal policy invariant
- Shaping with $$\phi \approx V^\pi$$ makes consequences of actions more immediate
- Shaping, and then ignoring all but the first term, gives policy gradient

### Variance Reduction

Previously showed that taking  

$$
\hat{A}_t = r_t, + r_{t+1} + \cdots - b(s_t)
$$

for any function $$b(s_t)$$, gives an unbiased policy gradient estimator.  

$$b(s_t) \approx V^\pi(s_t)$$ gives variance reduction.  

### Value Functions in the Future

_(reward-to-go)_  

Subtracting out baselines, get advantage estimators  

$$
\begin{align}
&\hat{A}_t^{(1)} = r_t + \gamma V(s_{t+1}) - V(s_t) \\
&\hat{A}_t^{(2)} = r_t + r_{t+1} + \gamma^2 V(s_{t+2}) - V(s_t) \\
&\dots \\
&\hat{A}_t^{(\infty)} = r_t + \gamma r_{t+1} + \gamma^2 r_{t+2} + \cdots  - V(s_t) \\
\end{align}
$$

$$\hat{A}_t^{(1)}$$ has low variance but high bias, $$\hat{A}_t^{(\infty)}$$ has high variance but low bias.  
Using intermediate $$k$$ gives an intermediate amount of bias and variance.  


---

# Assignments

Code could be found on [GitHub](https://github.com/smdsbz/CS294-assignment).  

## Assignment 1

For code, see [GitHub Repo](https://github.com/smdsbz/CS294-assignment/blob/master/hw1/WarmUp.ipynb).  

**Section 1: Getting Set Up**  

- Out-Dated Pre-Trained `.pkl`
    - Bit of hack: replace all `v1` seen, could be found in `expert/` and `demo.bash`, with `v2`.  
- `mujoco-py` Installation Related *(tested on Ubuntu 18.04)*
    - `GL/osmesa.h` not found
        - `sudo apt install libosmesa6-dev`
    - `-lGL` not found from `ld`
        - `sudo apt install libgl*` (`*` is the *nix wildcard, expand it yourself)
    - `patchelf` command not found
        - `sudo apt install pathelf`

**Section 2: Warmup**  

1. Dump the `expert_data` in `hw1/run_expert.py` to `pkl`, will be used as training data
2. Define network
    - Model: simple NN
    - Loss: MSE
    - Optimizer: SGD
3. Train

---

## Assignment 2

### Section 4: Implement Policy Gradient

For code, see [GitHub Repo](https://github.com/smdsbz/CS294-assignment/blob/master/hw2/train_pg.py)

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

---

## Assignment 3

For code, see [GitHub Repo](https://github.com/smdsbz/CS294-assignment/blob/master/hw3/dqn.py)

### Section 3: Implementation

**Bellman Error**  

1. Build two Q networks, the exploring policy and the update target respectively
2. Calculate state-value by multiplying the series of actions you take and their weights, i.e. Q
3. Calculate state-action-value by $$r + \gamma \max_{a'} Q$$
4. The Bellman error is the differenct in Q values between the exploring policy and target policy

The rest will be basic RTFM :smile:.  
