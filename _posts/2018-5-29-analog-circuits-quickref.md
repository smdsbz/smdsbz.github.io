---
layout: article
title: Analog Circuits Quick-Ref
tags: CourseNotes
---

Quick reference for Analog Circuits.  

{:toc}

---

### Triode Equations

#### Three States

- $$v_{CE}$$ from 0 to + :
    - Saturation
    - **Active**
- Cut-off (close to but still above zero)

#### Basic Relations

$$
\begin{align}
I_{CQ} &= \beta I_{BQ} \\
I_{EQ} &= I_{BQ} + I_{CQ} \\
\end{align}
$$

#### Internal Resistance

$$
\begin{align}
r_{be} &= r_{bb'} + (1+\beta)\frac{26\mathrm{mV}}{I_{EQ}(\mathrm{mA})} \\
       &= r_{bb'} + \frac{26\mathrm{mV}}{I_{BQ}(\mathrm{mA})} \\
\end{align}
$$

> **Note**  
> Default value for $$r_{bb'}$$ is $$200\Omega$$ .  

#### Amplifier Resistance

$$
\begin{align}
R_i &= \begin{cases}
R_{b'} \parallel [r_{be} + (1+\beta)R_e] &\text{(common-emitter)} \\
R_{b'} \parallel [r_{be} + (1+\beta)(R_e \parallel R_L)] &\text{(common-collector)} \\
R_e \parallel \frac{r_{be}}{1+\beta} &\text{(common-base)}
\end{cases} \\
R_o &= \begin{cases}
R_c &\text{(common-emitter)} \\
R_e \parallel \frac{r_{be} + (R_s \parallel R_{b'})}{1+\beta} &\text{(common-collector)} \\
R_c &\text{(common-base)}
\end{cases}
\end{align}
$$

#### Amplification

$$
\begin{align}
\dot{A}_V &= \frac{\dot{V}_o}{\dot{V}_i} &\text{(triode output - triode input)} \\
\dot{A}_{VS} &= \frac{\dot{V}_o}{\dot{V}_s}
           = \frac{R_i}{R_s+R_i}\dot{A}_V &\text{(triode output - source stimulation)} \\
\\
\dot{A}_V &= \prod_n \dot{A}_{V_{n}} \\
\end{align}
$$

> **Note**  
> Grab onto $$I_B$$ , since it contains relation between input segment and output segment.  

> **Note**  
> $$\dot{V}_O$$ is $$R_L$$-dependant.  
> When cascading directly, $$R_{i_n}=R_{L_{n-1}}$$ is introduced via contributing to $$\dot{V}_{O_{n-1}}$$ .  

---

### MOS Equations

#### Three States

- $$v_{DS} \, (=v_{GS}-V_P)$$ from 0 to + :
    - Variant resistance
    - **Saturation**
- Cut-off (almost 0)

#### Basic Relations

$$
\begin{align}
i_D &= K_n (v_{GS} - V_T)^2 \cdot (1 + \lambda v_{DS}) \\
g_m &= \frac{\partial{i_D}}{\partial{v_{GS}}}\vert_{v_{DS}} \\
    &= 2 K_n (v_{GS} - V_T) \\
    &= 2 \sqrt{K_{n/P}I_{DQ}} \\
    r_{ds} &= [\lambda K_n (v_{GS} - V_T)^2]^{-1} = \frac{1}{\lambda i_D} &\text{(usually large, often ignored)} \\
\end{align}
$$

$$
\begin{align}
\text{Where} \hspace{3em}
V_T &= \text{Threashold Voltage } . \\
\end{align}
$$

#### Amplifier Resistance

$$
\begin{align}
R_i &= \begin{cases}
R_{g1} \parallel R_{g2} &\text{(common-source)} \\
R_{g1} \parallel R_{g2} &\text{(common-drain)} \\
\frac{1}{g_m} \parallel R &\text{(common-gate)} \\
\end{cases}
\\
R_o &= \begin{cases}
R_d \parallel r_{ds} &\text{(common-source)} \\
\frac{1}{g_m} \parallel R_{(source)} \parallel r_{ds} &\text{(common-drain)} \\
R_d \parallel r_{ds} &\text{(common-gate)} \\
\end{cases} \\
\end{align}
$$

---

### Operational Amplifier

#### Subtraction

$$
\begin{align}
v_o &= - \frac{R_f}{R_n} v_{in} + (1 + \frac{R_f}{R_n}) v_{ip} \\
\end{align}
$$

> **Note**  
> Given condition of virtual-short and virtual-open, the resistance $$R_P$$ , the one before amplifier's positive input $$v_P$$ immediately, is silenced and of no influence on circuit output.  

---

### Feedback

#### Patterns

- Output
    - $$R_L$$ grounded → Shunt / Voltage sampling
    - $$R_L$$ not grounded → Series / Current sampling
- Input
    - Same as input → Shunt / Current feedback
    - Different from input → Series / Voltage feedback

#### Symbols

- $$\dot{F}$$ → feedback index 反馈系数
- $$\dot{A}_{XF}$$ → close-circuit amplification 闭环增益
- $$\dot{A}_{VF}$$ → close-circuit voltage amplification 闭环电压增益

Where $$X$$ could be one of $$V$$, $$I$$, $$R$$, $$G$$ accrodingly, w.r.t. feedback type.  

#### Commonly Used Approximation

Given conditions of *strong negative feedback*, i.e. $$\dot{A}$$ very large, have

$$
\begin{align}
\dot{A}_{XF} &= \frac{\dot{A}}{1 + \dot{A}\dot{F}} \approx \frac{1}{\dot{F}} \\
\end{align}
$$

---

### Signal Generating Circuits

#### RC Bridging Sine Oscillator

$$
\begin{align}
f_0 &= \frac{1}{2 \pi R C} \\
\end{align}
$$

Require $$A_V = 1 + \frac{R_f}{R_1} > 3$$, i.e. $$R_{f_{min}} = 2 R_1$$, where $$R_1$$ is grounding negative phase input of amplifier and $$R_f$$.  

---

### Inverted-Input Hysteresis Comparer

*Core concept: superposition principle*

$$
\begin{align}
V_{T-} &= \frac{R_1 V_{OL}}{R_1 + R_3} + \frac{R_3 V_{REF}}{R_1 + R_3} \\
V_{T+} &= \frac{R_1 V_{OH}}{R_1 + R_3} + \frac{R_3 V_{REF}}{R_1 + R_3} \\
\end{align}
$$

$$
\begin{align}
\text{Where} \hspace{3em}
R_1 &= \text{between reference voltage } V_{REF} \text{ and feedback resistance } R_3 \,, \\
R_2 &= \text{between input } v_i \text{ and interted input of amplifier } , \\
R_3 &= \text{between output } v_o \text{ and positive-phase input of amplifier } , \\
V_{REF} &= \text{reference voltage } .
\end{align}
$$

---

### Additional Notes

- **Exclude** $$R_L$$ when calculating output resistance $$R_o$$ , but do remember to **include** $$R_e$$ / $$R_s$$ (e.g. common-collector amplifier / common-source amplifier).
- **Include** $$R_L$$ when calculating output voltage $$\dot{V}_o$$ , for resistance converts $$I_C$$ to voltage.
- $$r_{bb'} \approx 200 \Omega$$ as default.
- When cascading,

    $$
    \begin{align}
    R_i &= R_{i_1} \\
    R_o &= R_{o_n} \\
    \end{align}
    $$

