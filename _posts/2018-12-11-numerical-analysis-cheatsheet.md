---
layout: article
title: Numerial Analysis CheatSheet
key: numerial-analysis-cheatsheet
tags: CourseNotes NumericalAnalysis
---

Cheatsheet for Numerical Analysis, may contain errors, open for comments / issues.  

<!-- more -->

[TOC]

# Interpolation

## Interpolation Polynomial

$$
p_{n}(x) = a_0 + a_1 x + a_2 x^2 + \cdots + a_n x^n
$$

where  

$$
\begin{cases}
p_n(x_0) = y_0 \\
p_n(x_1) = y_1 \\
\hspace{2em}\vdots \\
p_n(x_n) = y_n \\
\end{cases}
$$

s.t. $$y_i = f(x_i)$$.  

## Lagrange Interpolation

$$
L_n(x) = y_0 l_0(x) + y_1 l_1(x) + \cdots + y_n l_n(x)
$$

where  

$$
l_i(x) = \prod_{j = 0, j \neq i}^{n} \frac{x - x_j}{x_i - x_j}
$$

**Remainder**  

$$
R_n(x) = f(x) - L_n(x) = \frac{f^{(n+1)}(\xi)}{(n+1)!} \prod_{i=0}^{n}(x - x_i) \ , \hspace{1em} \xi \in (a, b)
$$

Or approximately  

$$
\vert R_n(x) \vert \leq \frac{M_{n+1}}{(n+1)!} \vert \prod_{i=0}^{n} (x - x_i) \vert
$$

where  

$$
\max_{a < x < b} \vert f^{(n+1)}(x) \vert = M_{n+1}
$$

**Special Case**  

- $$n = 1$$, linear interpolation
- $$n = 2$$, parabola interpolation

## Newton Interpolation

$$
\begin{align}
N_n(x) &= c_0 \cdot 1 \\
&+ c_1 (x - x_0) \\
&+ c_2 (x - x_0) (x - x_1) \\
& \hspace{2em} \cdots \\
&+ c_n (x - x_0) (x - x_1) \cdots (x - x_{n-1})
\end{align}
$$

where $$c_i = f[x_0, x_1, \cdots, x_i]$$ (see Sect. Difference Quotient).  

I.e.  

$$
N_n(x) = \sum_{i=0}^{n} \Big[ f[x_0, x_1, \cdots, x_i] \prod_{j=0}^{i} (x - x_j) \Big]
$$

### Difference Quotient

Having  

$$
f[x_i] = f(x_i) \ ,
$$

define  

$$
f[x_i, x_{i+1}, \cdots, x_{i+k}] = \frac{f[x_{i+1}, x_{i+2}, \cdots, x_{i+k}] - f[x_i, x_{i+1}, \cdots, x_{i+k-1}]}{x_{i+k} - x_i} \ .
$$

It can be concluded that  

$$
f[x_0, x_1, \cdots, x_k] = \sum_{i = 0}^{k} \frac{f(x_i)}{\prod_{j=0, j \neq i}^{k} (x_j - x_k)}
$$

**Remainder**  
$$
R_n(x) = f[x, x_0, x_1, \cdots, x_n] \prod_{i=0}^{n} (x - x_i)
$$

**Example**  

*==TODO==*  

## Hermite Interpolation

For $$n+1$$ different points $$\{ x_i \vert x_i \in [a, b], i = 0 \dots n \}$$, we have  

$$
f(x_i) = y_i \hspace{1em} f'(x_i) = y'_i \hspace{1em} i = 0, 1, \cdots, n \,.
$$

Form an interpolation polynomial $$H(x)$$ that has *no more than* $$2n + 1$$ *terms*, such that the following $$2n + 2$$ conditions are satisfied:  

$$
H(x_i) = y_i \hspace{1em} H'(x_i) = y'_i \hspace{1em} i = 0, 1, \cdots, n \,.
$$

The highest form, $$H_{2n+1}(x)$$, can be writen as follow:  

$$
H_{2n+1}(x) = \sum_{i = 0}^{n} \big[ y_i \alpha_i(x) + y'_i \beta_i(x) \big] \,,
$$

where  

$$
\begin{align}
\alpha'_i(x_j) &= 0 \\
\alpha_i(x_j) &= \delta_{ij} =
\begin{cases}
0 &j \neq i \\
1 &j = i
\end{cases} \,, \\
\\
\beta_i(x_j) &= 0 \\
\beta'_i(x_j) &= \delta_{ij} = 
\begin{cases}
0 &j \neq i \\
1 &j = i
\end{cases} \,.
\end{align}
$$

Both $$\alpha_i(x)$$ and $$\beta_i(x)$$ have no more than $$2n+1$$ terms, and the term $$(x - x_j)^2, j = 0, 1, \cdots, i-1, i+1, \cdots, n$$ is always included.  

**Solving Coefficients: $$\alpha$$, $$\beta$$**  

Define  

$$
l_i(x) = \prod_{j = 0, j \neq i}^{n} \frac{x - x_j}{x_i - x_j} \hspace{2em} \text{s.t.} \hspace{1em} l_i(x_j) = \begin{cases} 0 &j \neq i \\ 1 &j = i \end{cases}
$$

*==TODO: derivation process==*  

It therefore can be concluded that  

$$
\begin{align}
H_{2n+1}(x) &= \sum_{i = 0}^{n} \bigg\{ y_i \underbrace{ \Big[ 1 - 2(x - x_i) \sum_{j = 0, j \neq i}^{n} \frac{1}{x_i - x_j} \Big] l_i^2(x) }_{\alpha_i(x)} + y'_i \underbrace{(x - x_i)l_i^2(x)}_{\beta_i(x)} \bigg\} \\
\\
H_3(x) &= y_0 \alpha_0 \Big( \frac{x - x_0}{h} \Big) + y_1 \alpha_1 \Big( \frac{x - x_0}{h} \Big) \\
&+ h y'_0 \beta_0 \Big( \frac{x - x_0}{h} \Big) + h y'_1 \beta_1 \Big( \frac{x - x_0}{h} \Big) \hspace{2em} h = x_1 - x_0
\end{align}
$$

**Remainder**  
$$
\begin{align}
R_{2n+1}(x) &= \frac{f^{(2n+2)}(\xi)}{(2n + 2)!} \Big[ \prod_{i = 0}^{n} (x - x_i) \Big]^2 \hspace{2em} \xi \in (a, b) \\
&= \frac{f^{(2n+2)}(\xi)}{(2n + 2)!} \, \omega_{n+1}^2(x) \\
\\
\vert R_3(x) \vert &\leq \frac{h^4}{384} \max_{x_0 \leq x \leq x_1} \vert f^{(4)} (x) \vert \hspace{2em} h = x_1 - x_0
\end{align}
$$

**Example**  

*==TODO==*  

**Segmental Hermite Interpolation**  

Apply Hermite Interpolation for each segment s.t. $$S_3(x_i) = y_i, S_3'(x_i) = y_i'$$.  


# Integration

**Naive Approach**  

$$
\int_{a}^{b} f(x) dx \approx \begin{cases}
\sum_{i = 0}^{n} A_i f(x_i) &\text{(common)} \\
(b - a) \times \frac{f(a) + f(b)}{2} \\
(b - a) \times f\big( \frac{a + b}{2} \big) \\
(b - a) \times \frac{f(a) + 4f\big( \frac{a + b}{2} \big) + f(b)}{6} &\text{(Simpson)}
\end{cases}
$$

## Algebraic Precision

For $$f(x) = 1, x, x^2, \dots$$, if $$I(f)$$ holds when $$f(x) = x^k$$ and fails when $$f(x) = x^{k + 1}$$, $$I(f)$$ has algebraic precision of $$k$$.  

## Interpolation Approach

$$
\begin{align}
I &= \int_{a}^{b} f(x) dx \approx \int_{a}^{b} L_n(x) dx \\
&= \int_{a}^{b} \Big( \sum_{i = 0}^{n} f(x_i) l_i(x) \Big) dx \\
&= \sum_{i = 0}^{n} \Big[ \int_{a}^{b} l_i(x) dx \Big] f(x_i)
\end{align}
$$

### Newton-Cotes

$$
\begin{align}
\int_{a}^{b} f(x) dx &\approx \sum_{i = 0}^{n} \Big[ \int_{a}^{b} l_i(x) dx \Big] f(x_i) \\
&= (b - a) \sum_{i = 0}^{n} \Bigg[ \underbrace{\frac{\int_{a}^{b} l_i(x) dx}{b - a}}_{C_i, \, \text{Cotes Coeff.}} \cdot f(x_i) \Bigg]
\end{align}
$$

Let  

$$
\begin{align}
&h = \frac{b - a}{n} \\
&x_i = a + ih \\
\Rightarrow \ &\mathrm{d}x = h \, \mathrm{d}t
\end{align}
$$

Substitute into $$C_i$$, get  

$$
\begin{align}
C_i &= \frac{1}{nh} \int_{0}^{n} \Bigg[ \prod_{j = 0, j \neq i}^{n} \frac{a + th - (a + jh)}{a + ih - (a + jh)} \ h \, \mathrm{d}t\Bigg] \\
&= \frac{(-1)^{n - i}}{n \times i! \times (n - i)!} \ \int_{0}^{n} \Bigg[ \prod_{j = 0, j \neq i}^{n} (t - j) \Bigg] \ \mathrm{d}t
\end{align}
$$

Specially, when $$n = 4$$  

$$
\int_{a}^{b} f(x) \mathrm{d}x \approx (b - a) \times \frac{7 f(x_0) + 32 f(x_1) + 12 f(x_2) + 32 f(x_3) + 7 f(x_4)}{90} \hspace{2em} \text{(Cotes)}
$$

> **Note**  
> For $$n$$-th Newton-Cotes, and $$n$$ is even, algebraic precision reaches $$n + 1$$.  

**Remainder**  

- Trapezoid (algebraic precision: 1)  
  $$
  \begin{align}
  R_T &= \frac{h^{1 + 2}}{(1 + 1)!} \ \int_{0}^{1} f^{(1 + 1)}(\xi) (t - 0) (t - 1) \mathrm{d}t \\
  &= - \frac{h^3}{12} f''(\eta) \hspace{1em} \eta \in [a, b] \,, h = b - a
  \end{align}
  $$

- Simpson (algebraic precision: 2)  
  $$
  \begin{align}
  R_S &= \int_{a}^{b} \frac{f^{(4)}(\xi)}{4!} (x - a) \big( x - \frac{a + b}{2} \big)^2 (x - b) \mathrm{d}x \\
  &= - \frac{h^5}{90} f^{(4)}(\eta) \hspace{1em} h = \frac{b - a}{2}
  \end{align}
  $$

- Cotes (algebraic precision: 5)  
  $$
  R_C = - \frac{8 h^7}{945} \cdot f^{(6)}(\eta) \hspace{1em} h = \frac{b - a}{4}
  $$

**Multiply**  

- Multiple Trapezoid  
  $$
  \begin{align}
  I &= \frac{h}{2} \big[ f(a) + 2 \sum_{i = 1}^{n - 1} f(x_i) + f(b) \big] \\
  \\
  R_{T_n} &= \sum R_T = - \frac{n h^3}{12} f''(\eta) \hspace{1em} h = b - a
  \end{align}
  $$

- Multiple Simpson  
  $$
  \begin{align}
  I &= \sum_{i = 0}^{m - 1} \int_{x_{2i}}^{x_{2i + 2}} f(x) \mathrm{d}x \\
  &\approx \frac{2h}{6} \big[ f(a) + 4 \sum_{i = 0}^{m - 1} f(x_{2i + 1}) + 2 \sum_{i = 1}^{m - 1} f(x_{2i}) + f(b) \big] \hspace{1em} \text{s.t.} \ n = 2m \\
  \\
  R_{S_n} &= \sum R_S = - \frac{m h^5}{90} f^{(4)}(\eta) \hspace{1em} h = \frac{b - a}{2}
  \end{align} \\
  $$

- Multiple Cotes ($$n = 4m$$)  
  $$
  \begin{align}
  I &= \sum_{i = 0}^{m - 1} \int_{x_{4i}}^{x_{4i + 4}} f(x) \mathrm{d}x \\
  &\approx \frac{4h}{90} \Big[ 7 f(a) + 14 \sum_{i = 0}^{m - 1} f(x_{4i}) + 32 \sum_{i = 0}^{m - 1} f(x_{4i + 1}) + 12 \sum_{i = 0}^{m - 1} f(x_{4i + 2}) + 32 \sum_{i = 0}^{m - 1} f(x_{4i + 3}) + 7 f(b) \Big] \\
  \\
  R_{C_n} &= - \frac{m \cdot 8 h^7}{945} f^{(6)}(\eta) \hspace{1em} h = \frac{b - a}{4}
  \end{align}
  $$

> **Algebraic Precision**  
> 1 higher than non multiply version.  

**Variant Step Size**  

Half the step size, till $$\delta \leq \epsilon$$.  

**Romberg**  

$$
\begin{align}
\overline{T} &= T_{2n} + \frac{1}{3} (T_{2n} - T_{n}) = \frac{4}{3} T_{2n} - \frac{1}{3} T_n \\
\\
\frac{4}{3} T_{2n} &= \frac{4}{3} \cdot \frac{h / 2}{2} \Big[ f(a) + 2 \sum_{i =  n}^{n - 1} f(x_{i + \frac{1}{2}}) + 2 \sum_{i = 1}^{n - 1} f(x_i) + f(b) \Big] \\
&= \frac{h}{6} \Big[ 2 f(a) + 4 \sum_{i = 0}^{n - 1} f(x_{i + \frac{1}{2}}) + 4 \sum_{i = 1}^{n - 1} f(x_i) + 2 f(b) \Big] \\
\frac{1}{3} T_n &= \frac{h}{6} \Big[ f(a) + 2 \sum_{i = 1}^{n - 1} f(x_i) + f(b) \Big] \\
\\
\Rightarrow \overline{T} &= \frac{h}{6} \Big[ f(a) + 4 \sum_{i = 0}^{n - 1} f(x_{i + \frac{1}{2}}) + 2 \sum_{i = 1}^{n - 1} f(x_i) + f(b) \Big] = S_{2n}
\end{align}
$$

# Ordinary Differential Equation

$$
\begin{cases}
y' = f(x, y) \\
y(x_0) = y_0
\end{cases}
$$

## Euler Method

**Method 1: to Difference Quotient**  

$$
y'(x) = \lim_{h \rightarrow 0} \frac{y(x_n + h) - y(x_n)}{h} \approx \frac{y(x_{n+1}) - y(x_n)}{h}
$$

**Method 2: Integral**  

$$
\int_{x_n}^{x_{n+1}} y' \mathrm{d}x = \int_{x_n}^{x_{n+1}} f[x, y(x)] \mathrm{d}x
$$

Get  

$$
\begin{cases}
y_{n+1} = y_n + h f(x_n, y_n) \\
y_0 = y(x_0)
\end{cases}
$$

### Hidden Euler

$$
\begin{cases}
y_{n+1} = y_n + h f(x_{n+1}, y_{n+1}) \\
y_0 = y(x_0)
\end{cases}
$$

> **Note**  
> $$y_{n+1}$$ is unkown.  

### Two Step Euler

$$
\begin{cases}
y_{n+1} =  y_{n-1} + 2h f(x_n, y_n) \\
y_0 = y(x_0)
\end{cases}
$$

### Trapezoid Euler

$$
\begin{cases}
y_{n+1} = y_n + \frac{h}{2} \big[ f(x_n, y_n) + f(x_{n+1}, y_{n+1}) \big] \\
y_0 = y(x_0)
\end{cases}
$$

> **Note**  
> **Trapezoid Euler** = average of **Euler** and **Hidden Euler**  

## Regional Truncation Error

$$
T_{n+1} = y(x_{n+1}) - y_{n+1} \propto O(h^{p+1}) \hspace{1em} \text{s.t.} \ \text{algebraic precision of} \ p
$$

## Improved Euler Method

- propose: $$\overline{y}_{n+1} = y_n + h f(x_n, y_n)$$
- calibration: $$y_{n+1} = y_n + \frac{h}{2} \big[ f(x_n, y_n) + f(x_{n+1}, \overline{y}_{n+1}) \big]$$

## Runge-Kutta Method

$$
\begin{cases}
y_{n+1} = y_n + c_1 k_1 + c_2 k_2 \\
k_1 = h f(x_n, y_n) \\
k_2 = h f(x_n + a_2 h, y_n + b_{21} k_1)
\end{cases}
$$

- Second-Order Runge-Kutta: Improved Euler Method
- Third-Order Runge-Kutta: Kutta Euqations  
  $$
  \begin{cases}
  y_{n+1} = y_n + \frac{1}{6} k_1 + \frac{4}{6} k_2 + \frac{1}{6} k_3 \\
  k_1 = h f(x_n, y_n) \\
  k_2 = h f(x_n + \frac{1}{2} h, y_n + \frac{1}{2} k_1) \\
  k_3 = h f(x_n + h, y_n - k_1 + 2 k_2)
  \end{cases}
  $$









### Variable Step Size

For $$p$$-th order Runge-Kutta,  

$$
\Delta \approx \vert \frac{1}{2^p - 1} \big[ y_{n+1}^{(h/2)} y_{n+1}^{(h)} \big] \vert
$$

- while $$\Delta > \sigma$$, half the step size
    - ensure truncational error bellow threshold
- while $$\Delta < \sigma$$, double the step size
    - lesser computational cost & accumulated round-off error

## Convergability

==TODO:==  

## Stability

==TODO:==  

# Solving Equations

## Bisection Method

### Prior Estimate

For series of ranges contianing root $$[a_0, b_0] \supset [a_1, b_1] \supset \cdots \supset [a_n, b_n] \supset \cdots$$, have  

$$
b_n - a_n = \frac{1}{2} (b_{n - 1} - a_{n - 1})
$$

Thus, the following inequality holds  

$$
\vert x_n - x^* \vert \leq \frac{b_n - a_n}{2}
$$

Want  

$$
\vert x_n - x^* \vert \leq \epsilon
$$

## Iteration Method

Construct equation, such that  

$$
x^* = \phi(x^*)
$$

Have iteration  

$$
x_{n+1} = \phi(x_n) \hspace{2em} n = 0, 1, \cdots
$$

> **Note**  
> A convergent iteration is required!  

**Convergence**  

- $$\phi(x)$$ is (1st order) differentiable in range $$[a, b]$$.
- For $$x \in [a, b]$$, $$\phi(x) \in [a, b]$$.
- Exists $$0 < L < 1$$, s.t. $$\forall x \in [a, b]$$,  

  $$
  \vert \phi'(x) \vert \leq L < 1 \ .
  $$


Have $$x = \phi(x)$$, for $$x \in [a, b]$$, exists one single root $$x^*$$. And for arbitrary initial start-off value $$x_0 \in [a, b]$$, $$\lim_{n \rightarrow \infty} x_n = x^*$$.  
