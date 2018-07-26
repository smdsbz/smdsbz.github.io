---
layout: article
title: Signal and System Notes
tags: CourseNotes
---

Preparation notes for upcomming Signal and System test.  

{:toc}

## Bulbs

1.  Get edges of image

    $$
    \begin{align}
    h'(t) &= \delta(t) * h'(t) \\
          &= \delta'(t) * h(t)
    \end{align}
    $$

## Examples

**Question**  
All possible original functions in t-domain for given Laplace transform $$F(s) = \frac{3s^2-30s+74}{(s-4)(s-5)(s-6)}$$.  

**Solution**  
$$F(s)$$ could be reduced to

$$
F(s) = \frac{1}{s-4} + \frac{1}{s-5} + \frac{1}{s-6}    \,.
$$

Therefore,

$$
\begin{align}
f(t) = \begin{cases}
    (e^{4t}+e^{5t}+e^{6t})u(t),         &\mathrm{Re}[s] \gt 6 \\
    (e^{4t}+e^{5t})u(t) - e^{6t}u(-t),  &5 \lt \mathrm{Re}[s] \lt 6 \\
    e^{4t}u(t) - (e^{5t}+e^{6t})u(-t),  &4 \lt \mathrm{Re}[s] \lt 5 \\
    -(e^{4t}+e^{5t}+e^{6t})u(-t),       &\mathrm{Re}[s] \lt 4
    \end{cases}
\end{align}
$$

> **Note**  
> Denote the norm of the only singular point of $$F(s)$$ as $$v$$. Within convergence domain, i.e. $$\vert s \vert \gt v$$, have
> 
> $$
> \mathcal{L}^{-1}\{F(s)\} = f(t)u(t) \,.
> $$
> 
> Out of convergence domain, i.e. $$\vert s \vert \lt v$$, get corresponding left-sided sequence
> 
> $$
> \mathcal{L}^{-1}\{F(s)\} = -f(t)u(-t)   \,.
> $$


**Question**  
Given the Laplace transform of a causal system $$F(s)=\frac{s^2+3s+2}{s^2+4s+4}$$, $$f(0^+)=\underline{\>-1\>}$$, $$f(+\infty)=\underline{\>0\>}$$.  

**Solution**  

$$
\begin{align}
F(s) &= \frac{s+1}{s+2}     \\
     &= 1 - \frac{1}{s+2}   \,.
\end{align}
$$

Therefore,

$$
f(t) = \delta(t) - e^{-2t}u(t)  \,,
$$

and get $$f(0^+) = -1$$, $$f(\infty) = 0$$.  


**Question**  
Given the differential equation of a discrete causal system as $$y(k) - \alpha y(k-1) = e(k)$$, where $$\alpha \in \mathbb{R}$$.  
1.  $$H(z)$$ and its convergence domain, unit impulse response $$h(k)$$.  
2.  Range of $$\alpha$$ which allow system to be stable.  
3.  For $$\alpha=\frac{1}{2}$$, $$y(-1)=4$$, $$e(k)=0$$, solve system response $$y(k)\;(k\geq0)$$.  

**Solution**  
1.  Apply Z-Transform on both sides, get

    $$
    \begin{align}
                  &Y(z) - \alpha \frac{1}{z} Y(z) = E(z) \\
    \Rightarrow\; &(1 - \alpha \frac{1}{z}) Y(z) = E(z) \\
    \Rightarrow\; &H(z) = \frac{Y(z)}{E(z)} = \frac{1}{1 - \alpha z^{-1}} = \frac{z}{z - \alpha} \,,
    \end{align}
    $$

    where $$\vert \alpha \vert \lt \vert z \vert \lt 1$$.  

    > **Note**
    > $$\vert z \vert \lt 1$$ is given by the conditon of convergence of the Z-Transform $$v^ku(t) \leftrightarrow \frac{z}{z-v}$$.

    Unit impulse response is $$h(k) = \alpha^k u(t)$$.  
2.  Having $$\vert \alpha \vert \lt \vert z \vert \lt 1$$, must have $$\vert \alpha \vert \lt 1$$.  
3.  Assume system response, i.e. zero-input response in this case, have the format $$y(k) = c \alpha^k u(k)$$.  
    Substitute with given constraints, get $$y(k) = (\frac{1}{2})^{k-1}u(t)$$.  

**Question**  
Given the Fourier transform of signal $$f(t)$$ to be $$F(j\omega)$$, the Fourier transform of $$g(t)=e^{j4t}f(2t-4)$$ should be $$\underline{\> \frac{1}{2} F(j\frac{1}{2}(\omega-4)) e^{-j2(\omega-4)} \>}$$.  

**Solution**  

$$
\begin{align}
g(t) &= e^{j4t}f(2(t-2)) \\
\end{align}
$$

Break down step-by-step

$$
\begin{align}
f(2t) &\leftrightarrow \frac{1}{2} F(j\frac{1}{2}\omega) \\
f(2(t-2)) &\leftrightarrow \frac{1}{2} F(j\frac{1}{2}\omega) e^{-j2\omega} \\
e^{j4t}f(2(t-2)) = g(t) &\leftrightarrow \frac{1}{2} F(j\frac{1}{2}(\omega-4)) e^{-j2(\omega-4)} \\
\end{align}
$$

**Question**  

$$
\underline{\> x(k) = \begin{cases}
\begin{align}
k, \hspace{1em} &k=1, 2 \\
0, \hspace{1em} &otherwise
\end{align}
\end{cases} \>} \leftrightarrow X(z) = z^{-1} + 2z^{-2} \\
$$

**Solution**  

Knowing $$\mathcal{Z}\{\delta(k)\} = 1$$ and $$\mathcal{Z}\{f(k-n)\} = z^{-n}F(z)$$, combined have

$$
x(k) = \delta(k-1)\cdot1 + \delta(k-2)\cdot2 \leftrightarrow X(z) = z^{-1}\cdot1 + z^{-2}\cdot2
$$


**Question**  

System $$H(s) = \frac{e^s}{s+1},\,\mathrm{Re}\{s\}\gt-1$$ is <u>&nbsp;non-causal stable system&nbsp;</u>.  

**Solution**  

Knowing $$\mathcal{L}\{e^{\alpha t}u(t)\} = \frac{1}{s-\alpha}$$ and $$\mathcal{L}\{u(t-\tau)\} = \frac{1}{s}e^{-\tau s}$$, combined have

$$
h(t) = e^{-(t+1)}u(t+1) \leftrightarrow H(s) \,. \\
$$

Obviously the system converges.  
Noticing that non-zero value spans to $$t\lt0$$, therefore the system is non-causal.  


**Question**  

Given $$h_1(t)=\delta(t-1)$$, $$h_2(t)=e^{-2t}u(t-2)$$, $$h_3(t)=e^{-t}u(t)$$, and $$r(t) = e(t) * [h_1(t) + 1] * [h_2(t) + h_3(t)]$$.  
1.  Unit impulse response $$h(t)$$.  
2.  Causal? Stable? Reasons!  

**Solution**  

1.  Knowing $$e^{\alpha t}u(t) \leftrightarrow \frac{1}{s-\alpha}$$ and $$f(t-t_c) \leftrightarrow F(s)e^{-st_c}$$ (delay), have

    $$
    \begin{align}
    H_1(s) &= e^{-s} \,, \\
    H_2(s) &= \mathcal{L}\{e^{-2(t-2)-4}u(t-2)\} \\
           &= e^{-4}\frac{1}{s+2}e^{-2s} \\
           &= e^{-2(s+2)}\frac{1}{s+2} \> (\mathrm{Re}\{s\}\gt-2) \,, \\
    H_3(s) &= \frac{1}{s+1} \> (\mathrm{Re}\{s\}\gt-1) \,. \\
    \end{align}
    $$

    Therefore,

    $$
    \begin{align}
    R(s) &= E(s) \cdot (1+e^{-s}) \cdot (e^{-2(s+2)}\frac{1}{s+2} + \frac{1}{s+1}) \\
         &= E(s) \cdot (\frac{1}{s+2}e^{-2s-4} + \frac{1}{s+1} + \frac{1}{s+2}e^{-3s-4} + \frac{1}{s+1}e^{-s}) \,.
    \end{align}
    $$

    Back in t-domain,

    $$
    \begin{align}
    r(t) &= e(t) * \Big[ e^{-4}e^{-2(t-2)}u(t-2) + e^{-t}u(t) + e^{-4}e^{-2(t-3)}u(t-3) + e^{-(t-1)}u(t-1) \Big] \\
    \Rightarrow h(t) &= e^{-t}u(t) + e^{-(t-1)}u(t-1) + e^{-2t}u(t-2) + e^{-2(t-1)}u(t-3) \,. \\
    \end{align}
    $$

2.  $$h(t)=0,\, t\lt0$$, therefore causal.  
    System function $$H(s)$$ converges for all $$j\omega$$, therefore stable.  






----------------------------------------


## Transforms

### Fourier Transform

#### Frequently Seen Pattern

$$
\begin{align}
\mathcal{F}\{\delta(t)\} &= 1 \\
\mathcal{F}\{1\} &= 2 \pi \delta(\omega) \\
\mathcal{F}\{u(t)\} &= \pi\delta(\omega) + \frac{1}{j\omega} \\
\mathcal{F}\{e^{-\alpha t} u(t)\} &= \frac{1}{\alpha + j\omega} \\
\mathcal{F}\{t e^{-\alpha t} u(t)\} &= \frac{1}{(\alpha + j\omega)^2} \\
\mathcal{F}\{\cos\omega_ct\} &= \pi [ \delta(\omega+\omega_c) + \delta(\omega-\omega_c) ] \\
\mathcal{F}\{\sin\omega_ct\} &= j\pi [ \delta(\omega+\omega_c) + \delta(\omega-\omega_c) ] \\
\mathcal{F}\{G_\tau(t)\} = \mathcal{F}\{u(t+\frac{\tau}{2})-u(t-\frac{\tau}{2})\}
        &= \tau\text{Sa}\Big(\tau\frac{\omega}{2}\Big) = \tau\frac{\sin(\tau\omega/2)}{\tau\omega/2} \\
\mathcal{F}\{\text{Sa}\Big(\frac{\Omega t}{2}\Big)\} &= \frac{2\pi}{\Omega}\Big[u(\omega+\frac{\Omega}{2})-u(\omega-\frac{\Omega}{2})\Big] \\
\mathcal{F}\{\delta_T(t)\} = \mathcal{F}\{\sum_{n=-\infty}^{+\infty}\delta(t-nT)\}
&= \Omega\delta_{\Omega}(\omega) \hspace{1em} (\Omega = \frac{2\pi}{T}) \\
\end{align}
$$

#### Attributes

##### Symmetry Attribute

For Fourier transform

$$
f(t) \leftrightarrow f(j\omega) = R(\omega) \,, \\
$$

have

$$
R(t) \leftrightarrow 2 \pi f(\omega) \,, \\
$$

or

$$
\frac{1}{2 \pi} R(t) \leftrightarrow f(\omega) \,. \\
$$

##### Derivative Attribute

$$
\frac{d^nf(t)}{dt^n} \leftrightarrow (j\omega)^nF(j\omega) \\
$$


### Laplace Transform

#### Frequently Seen Pattern

$$
\begin{align}
\mathcal{L}\{\delta(t)\} &= 1      \\
\mathcal{L}\{u(t)\} &= \frac{1}{s}  \\
\mathcal{L}\{t^n e^{\alpha t} u(t)\}
        &= \frac{n!}{(s-\alpha)^{n+1}} \\
\mathcal{L}\{u(t+\tau)\} &= \frac{1}{s}e^{\tau s} \\
\mathcal{L}\{(t+\tau)^n e^{\alpha(t+\tau)} u(t+\tau)\}
        &= \frac{n!}{(s-\alpha)^{n+1}}e^{\tau s} \\
\end{align}
$$

#### Attributes

##### Derivative Attribute in t-domain

$$
\mathcal{L}\{\frac{d^nf(t)}{dt^n}\} = s^nF(s) - \sum_{i=0}^{n-1} s^{n-i-1} f^{(i)}(0^-) \,, \\
$$

in causal systems,

$$
\mathcal{L}\{\frac{d^nf(t)}{dt^n}\} = s^nF(s) \,. \\
$$



### Z-Transform

#### Discrete System

##### Differential Equation and Characteristic Equation

For a differential equation, with constant coefficiency,

$$
y(k+n)+ a_{n-1}y(k+n-1) + \cdots + a_0y(k) = 0  \,,
$$

introducing the *shifting operator* $$S$$, equation above could be denoted as

$$
(S^n + a_{n-1}S^{n-1} + \cdots + a_0) = 0   \,,
$$

from which you can get the corresponding characteristic equation

$$
S^n + a_{n-1}S^{n-1} + \cdots + a_0 = 0     \,.
$$

Assuming that $$v_1$$, $$v_2$$, $$\cdots$$, $$v_n$$ are characteristic roots, or eigenvalues, of the characteristic equation above, the equation could also be written in the following format

$$
(S-v_1)^m (S-v_{m+1}) \cdots (S-v_n) = 0    \,,
$$

where $$v_1$$ to $$v_m$$ are repeated roots.  
Then the zero-input response is

$$
y_{zi}(k) = (c_1 + c_2k + \cdots + c_mk^{m-1})v_1^k
            + c_{m+1}v_{m+1}^{k} + \cdots + c_nv^k  \,.
$$

##### Left-Sided and Right-Sided Sequence

$$
\begin{cases}\begin{align}
\text{Right-sided:}\> &f(k)u(k) \\
\text{Left-sided:}\>  &-f(k)u(-k-1) \\
\end{align}\end{cases}
$$

**Be very careful to the $$1$$ offset!**  


#### Frequently Used Pattern

$$
\begin{align}
\mathcal{Z}\{\delta(k)\} &= 1 \\
\mathcal{Z}\{v^ku(k)\} &= \frac{z}{z-v} \\
\mathcal{Z}\{ku(k)\} &= \frac{z}{(z-1)^2} \\
\mathcal{Z}\{kv^{k-1}u(k)\} &= \frac{z}{(z-v)^2} \\
\end{align}
$$

#### Attributes

##### Time Shifting (Delaying) Attribute

$$
\begin{align}
\mathcal{Z}\{f(k+1)\} &= \mathcal{Z}\{Sf(k)\} \\
        &= z[F(z) - f(0)] \,,
\end{align}
$$

or more general

$$
f(k+n) \leftrightarrow z^n \Big[ F(z) - \sum_{i=0}^{n-1} f(i) z^{-1} \Big] \qquad n \gt 0 \,.
$$

If all zero-states are $$0$$, e.g. in a causal system, could be reduced to

$$
f(k-n) \leftrightarrow z^{-n}F(z)   \,.
$$

