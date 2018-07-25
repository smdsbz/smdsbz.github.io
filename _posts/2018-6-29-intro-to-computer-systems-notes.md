---
layout: post
author: smdsbz
title: Introduction to Computer Systems Notes
---

{:toc}

# Key Concepts

## Computer Parts

-   控制部件
-   执行部件


## Machine Representation

### Floating-Points

_(IEEE 754 based)_

-   32-bit single precision
    -   1:S | 8:Exponent | 23:Significand
-   64-bit double precision
    -   1:S | 11:Exponent | 52:Significand

#### Norms

-   32-bit single precision
    -   $$(-1)^{\text{S}} \times (\overline{1. \text{Significand}}) \times 2^{\text{Exponent} - 127}$$
-   64-bit double precision
    -   $$(-1)^{\text{S}} \times (\overline{1. \text{Significand}}) \times 2^{\text{Exponent} - 1023}$$

#### Denorms

>   Exponent is zero, Significand non-zero.  

-   32-bit single precision
    -   $$(-1)^{\text{S}} \times (\overline{0. \text{Significand}}) \times 2^{-126}$$
-   64-bit double precision
    -   $$(-1)^{\text{S}} \times (\overline{0. \text{Significand}}) \times 2^{-1022}$$

#### Zero

>   Exponent is zero, Significand is zero.  

-   Positive: `0 000... 000...`
-   Negative: `1 000... 000...`

#### Infinity

>   Exponent all `1`, Significand is zero.  

-   Positive: `0 111... 000...`
-   Negative: `1 111... 000...`

#### NaN

>   Exponent all `1`, Significand is non-zero.  

-   Quiet: `? 111... 1??...`
-   Signaling: `? 111... 0??...`


## Instructions

### 微指令

&emsp;&emsp;微指令是微程序级命令，属于 **硬件范畴**。  

&emsp;&emsp;*控制部件* 通过 *控制线路* 向 *执行部件* 发出各种控制指令（即 *微指令*）。  

### 机器指令

&emsp;&emsp;机器指令处于 **硬件与软件的交界面**。  

&emsp;&emsp;若干条 *微指令* 可以构成一个 *微程序*，而一个微程序就对应了一个 *机器指令*。  

&emsp;&emsp;微程序固化在 CPU 内部的控制存储器中，机器运行时只读不写。微程序规定了电路级的工作方式。  

-   操作码 | 寻址方式 | 寄存器编号 | 立即数（位移量）

### 伪指令 / 宏指令

&emsp;&emsp;伪指令是由若干条 *机器指令* 组成的指令序列，属于 **软件范畴**。  


## Compilation

1.  C source file (`hello.c`) to *preprocessor*, get preprocessed source (`hello.i`)
    -   `gcc -E` flag
2.  Preprocessed source (`hello.i`) to *compiler*, get assembly source (`hello.s`)
    -   `gcc -S` flag
3.  Assembly source (`hello.s`) to *assembler*, get relocatable object file (`hello.o`)
4.  Object file (`hello.o`) and precompiled libraries (e.g. `printf.o`) to *linker*, get binary executable (`hello`)


## Binary File Conventions

### ELF (Executable and Linkable File)

0.  ELF header section
    -   包含文件结构说明信息
1.  `.text`: code section
2.  `.rodata`: read-only data section
3.  `.data`: initialized data section
4.  `.bss`: uninitialized data section
5.  `.symtab`: symbol table section
6.  `.rel.txt`: code relocation info section
7.  `.rel.dat`: data relocation info section
8.  `.debug`: debug info section
    -   `gcc -g` flag
9.  `.strtab`
10. `.line`
11. Section header table
    -   由若干个表项组成，每个节都有一个表项与之对应，每个表项描述对应的一个节的信息

### Executable

0.  ELF header section
1.  Program header section
3.  `.init`: initialization section
4.  `.text`: code section
5.  `.rodata`: read-only data section
6.  `.data`: initialized data section
7.  `.bss`: uninitialized data section
8.  `.symtab`: symbol table section
9.  `.rel.txt`: code relocation info section
10. `.rel.dat`: data relocation info section
11. `.debug`: debug info section
12. `.strtab`
13. `.line`
14. Section header table

In which, 0~5 are read-only, 6~7 are read-write, rest are not loaded during runtime.  


## Symbols

### Classification

-   Global Symbols
    -   Non-static function and non-static global variables.  
-   Local Symbols
    -   Static functions and global variables.  
    >   **Note**  
    >   Local variables are defined during runtime, stored temporarily in **stack**. They are **NOT** local symbols, and **NOT** linker's concern.  

-   External Symbols
    -   Global symbols defined externally, i.e. defined in other modules but referenced by this module.  

### Symbol Resolution

&emsp;&emsp;也称 *符号绑定*，是将 *引用符号* 与 *定义符号* 建立关联的过程。  

#### 全局符号的解析

##### 强 / 弱特性

-   强符号定义
    -   函数名和已初始化的全局变量名
-   弱符号定义
    -   未初始化的全局变量名

>   **Note**  
>   局部符号无强弱特性  

##### 链接器对多重定义符号的处理规则

1.  强符号不能多次定义
2.  若一个符号被定义为一次强符号和多次弱符号，则按强定义为准
3.  若有多个弱符号定义，则任选其中一个

#### Procedure

Linker maintains three sets, namely  

-   E: target files that will be merged into the final executable
-   U: unresoluted references of symbols
-   D: resoluted symbols

**Example**  

```bash
gcc -static -o myfunc func.o libx.a liby.a libz.a
```

>   **Note**  
>   Order counts! First define, then reference. Otherwise, there will be symbols left in the **U** set, causing linker error.  

### Relocation / 重定位

$$
\begin{align}
\text{Relocation Value} &= \text{Target Addr.} - \text{PC} \,, \\
\\
\text{where} \>
\text{PC} &= \text{Addr. of the next instruction} \\
          &= \text{addr. of section .text} + \text{addr. of current instruction} - \text{init} \,. \\
\end{align}
$$

**Example**  

Assuming `<main>` is at `0x08048380`, `<swap>` at `0x08048394`.  

```assembly
00000000 <main>:
...
6:          e8 fc ff ff ff      call <main+0x7>
...
```

After linking,  

```assembly
08048380 <main>:
...
 8048386:   e8 09 00 00 00      call 8048394
...
```

---

# Methodology

## Integer Division

**Example**  

$$
-14 / 4 = -3
$$

**Solution**

$$
\begin{align}
&\because   4_{10} = 100_{2} \,, \\
&\therefore K := 2 \,, \\
&\therefore (-14 + (2^K - 1)) \gg 4 = -3 \,.
\end{align}
$$

I.e.  

```
1111_0010 + 0000_0011 = 1111_0101
1111_0101 >> 2 = 1111_1101 = -3
```

## Integer Multiplication

-   Break down to additions

## Floating-Point Addition

1.  求阶差
2.  对阶
    -   小阶向大阶看齐
    -   为保证精度，保留部分尾数为 *附加位*
3.  尾数加减
    - [IEEE 754] 定点源码小数加减，带附加位
4.  规格化
    -   $$\pm 0.\text{bbb}\dots \> \text{and} \> \pm 1\text{b}.\text{bbb}\dots \Rightarrow \pm 1.\text{bbb}\dots$$
5.  尾数舍入
6.  若运算结果尾数为 0，则需要将阶码也置 0

## Floating-Point Multiplication / Division

-   尾数相乘 / 除，阶码相加减


---

# Special Notes

-   In the C programming language...
    -   when using the `<<` and `>>` operators, *logical shift* is applied to `unsigned`-s, *arithmetic shift* is applied to `signed`-s.
    -   when casting between types of different bit length, *bit extension / cut-off* will take place.
        -   *sign-bit extension* is applied to `unsigned`-s, whilst *0 extension* is applied to `signed`-s.

