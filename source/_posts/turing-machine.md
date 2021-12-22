---
title: 计算的数学模型——Turing机
tags: ["algorithm", "math"]
categories: ["algorithm"]
reward: true
copyright: true
date: 2021-12-22 13:00:00
thumbnail: 
---



本文是《近世计算理论导引：NP难度问题的背景、前景及其求解方法研究》的第一章《计算的数学模型——Turing机》的笔记。

<!--more-->

# Turing机的定义

Turing机由有穷个字符构成的字符表 $\{s_1, \cdots, s_n \}$、有穷个内部状态构成的内部状态表 $\{q_1,\cdots,q_v\}$ 和有穷条规则构成的行为准则集组成。

在Turing机的运行方式是对一个左右无穷的条带进行处理。该条带上有穷个方格被填入字符表 $\{s_1, \cdots, s_n \}$ 中的某一个字符，其余方格全部填入空白字符 $s_0$。内部状态 $q_i$ 指向着条带上的某一个方格。

初始时刻，将内部状态 $q_1$ 指向条带最左非 $s_0$ 方格的左边一个方格。

$$
\begin{matrix}
\cdots & s_0 & s_\lambda & \cdots & s_\mu & s_0 & \cdots \\
& \uparrow \\
& q_1
\end{matrix}
$$

Turing机后续的行为由有穷条规则构成的集合（行为准则集）指挥。其行为规则由以下四种类型，其外形为四元组。

$$
\begin{matrix}
\begin{cases}
q_i s_j s_k q_l \\
q_i s_j L q_i \\
q_i s_j R q_l
\end{cases} \\
0 \le j,k \le n \\
1 \le i,l \le v
\end{matrix}
$$

1. 第一条规则 $q_i s_j s_k q_l$ 指出：当Turing机当前的内部状态为 $q_i$，且指向的方格的字符为 $s_j$ 时，指针不移动，但将该方格的字符改为 $s_k$，并将内部状态改为 $q_l$。

$$
\begin{matrix}
\begin{matrix}
\cdots & s_j & \cdots \\
& \uparrow \\
& q_i
\end{matrix}
&\to&
\begin{matrix}
\cdots & s_k & \cdots \\
& \uparrow \\
& q_l
\end{matrix}
\end{matrix}
$$



2. 第二条规则 $q_i s_j L q_i$ 指出：当Turing机当前的内部状态为 $q_i$，且指向的方格的字符为 $s_j$ 时，指针向左移动一个方格，并将内部状态改为 $q_l$。

$$
\begin{matrix}
\begin{matrix}
\cdots & s_\lambda & s_j & s_\mu & \cdots \\
&& \uparrow \\
&& q_i
\end{matrix}
&\to&
\begin{matrix}
\cdots & s_\lambda & s_j & s_\mu & \cdots \\
& \uparrow \\
& q_l
\end{matrix}
\end{matrix}
$$



3. 第三条规则 $q_i s_j R q_i$ 指出：当Turing机当前的内部状态为 $q_i$，且指向的方格的字符为 $s_j$ 时，指针向右移动一个方格，并将内部状态改为 $q_l$。

$$
\begin{matrix}
\begin{matrix}
\cdots & s_\lambda & s_j & s_\mu & \cdots \\
&& \uparrow \\
&& q_i
\end{matrix}
&\to&
\begin{matrix}
\cdots & s_\lambda & s_j & s_\mu & \cdots \\
&&& \uparrow \\
&&& q_l
\end{matrix}
\end{matrix}
$$



4. 当没有任何一条规则匹配时，计算终止，称为**停机**。



一种Turing机的写法是：

$$
\begin{pmatrix}
\{ s_1, s_2\}, & \{ q_1, q_2,q_3\}, &
\begin{Bmatrix}
q_2 & s_0 & s_1 & q_3 \\
q_3 & s_2 & L & q_2 \\
q_1 & s_1 & s_0 & q_2
\end{Bmatrix}
\end{pmatrix}
$$


在任一给定时刻，Turing机带上的每一个方格内是什么字符，指针指向哪个方格，这些信息称为**格局/带格局**。



# Turing机所计算的函数

设有一个字符表 $A= \{ s_1, \cdots, s_n \}$，函数 $f$ 是从 $A^{*m}$ 到 $A^*$ 的一个部分函数，函数 $f$ 在某个子集上有定义，而在其余集上无定义。其中 $A^* = \bigcup_{k=1}^\infty A^k$ 的元素称为字，$A^{*m}$ 的元素为 $m$ 个字。

设有一个Turing机 $T$ 的字符表为 $\{s_1, \cdots, s_N\}$、内部状态表为 $\{ q_1, \dots, q_v \}$，Turing机 $T$ 实现了对函数 $f$ 的计算是指满足以下条件：

+ 若以 $\begin{matrix} s_0 & x_1 & s_0 & x_2 & s_0 & \cdots & s_0 & x_m  \\ \uparrow \\ q_1 \end{matrix}$ （其中 $x_i \in A^*$）为格局开始则
  + 最终 $T$ 会停机当且仅当 $f(x_1, \cdots, x_m)$ 有定义。
  + $T$ 停机时的构形 $\begin{matrix} s_0 & \times & \cdots & \times & \cdots & \times & s_0 \\ &&& \uparrow \\ &&& q_i \end{matrix}$ 中间忽略掉所有的 $s_0, s_{n+1}, \cdots, s_N$ 后剩下的字符串即是字 $f(x_1, \cdots, x_m)$。

满足上述条件的情况下, $f$ 被称为Turing机 $T$ 所计算的 $m$ 元函数，或称 Turing 机 $T$ 计算了 $m$ 元函数 $f$。

如果 $T$ 计算了 $f$，且满足以下两个附加条件，则称 $T$ 严格计算了 $f$：

1.  $N=n$，即 $T$ 没有使用 ${s_0} \bigcup A$ 以外的字符；
2. $T$ 停机时，带上的构形如 $\begin{matrix} \cdots & s_0 & s_0 & y & s_0 & s_0 & \cdots \\ &&& \uparrow \\ &&& q_i \end{matrix}$ ，其中字符串 $y$ 不包含空白 $s_0$, 即 $y \in A^*$。

目前已经证明：任一给定的Turing机计算的函数，一定能被某一个 Turing 机严格计算。



# Turing机所接受的语言

设Turing机 $T$ 的内部状态表为 $\{ q_1, \dots, q_v \}$， 字符表为 $A = \{s_1, \cdots, s_n\}$。

对于某个字 $u \in A^*$，如果以格局 $\begin{matrix} s_0 & u \\ \uparrow \\ q_1 \end{matrix}$ 开始 $T$ 会停机，则称 $T$ 接受字 $u$。

$T$ 所接受的语言 $L$ 是 $T$ 所接受的 $A^*$ 中的字的全体所构成的集合，$L \in A^*$。



# 计算复杂度

设函数 $f$ 为 Turing机 $T$ 所计算的 $m$ 元函数，设 $f(x_1, \cdots, x_m)$ 有定义，于是当以格局 $\begin{matrix} s_0 & x_1 & s_0 & x_2 & s_0 & \cdots & s_0 & x_m  \\ \uparrow \\ q_1 \end{matrix}$ 开始时 $T$ 最终会停机。

从开机到停机这一过程中所执行的四元组的次数称为**计算的步数**，也称为 $T$ 计算函数值 $f(x_1, \cdots, x_m)$ 的**时间复杂度**。

计算过程中带上最左一个曾经不为 $s_0$ 的格子到最右一个不曾为 $s_0$ 的格子之间是 $f$ 的 “工作部分”，之间所包含的格子数称为 $T$ 计算函数值 $f(x_1, \cdots, x_m)$ 的**空间复杂度**。

时间复杂度和空间复杂度统称**时空开销**。



# Church-Turing 论题

设任意给定一个字符表 $A = \{ s_1, \cdots, s_n\}$，$n \ge 1$，$f$ 是从 $A^{*m}$ 到 $A^*$ 上的一个任给的全函数，$m \ge 1$。

论题给出如下断言：

> 只要世上有一台机器能够实现对函数 $f$ 的计算，则一定存在一台 Turing 机能实现对 $f$ 的计算。

该论题目前已被证明永远无法证实，但尚未保证永远不能被证伪。



# Turing机的编码

给定一台Turing机：

$$
\begin{pmatrix}
\{ s_1, s_2\}, & \{ q_1, q_2,q_3\}, &
\begin{Bmatrix}
q_2 & s_0 & s_1 & q_3 \\
q_3 & s_2 & L & q_2 \\
q_1 & s_1 & s_0 & q_2
\end{Bmatrix}
\end{pmatrix}
$$

规定将自然数使用二进制书写，如 $s_2$ 写成 $s_{10}$，然后将Turing机的所有部分自然拉长写成 $\{ s, q, 0, 1, R, L \}^*$ 上的一个字，如：

$$
s1s10q1q10q11q10s0s1q11q11s10Lq10q1s1s0q10
$$

对 $\{s,q,0,1,R,L\}*$ 上的所有字使用字典序进行编号，可以将每一个Turing机 $T$ 对应上一个自然数，称为该Turing机的**字码** $M(T)$。

性质：

+ 给定一台 Turing机 $T$ 可以算出字码 $M(T) \in \mathbb{N}$。
+ 若 $T_1 \neq T_2$， 则 $M(T_1) \neq M(T_2)$。
+ $\forall v \in \mathbb{N}$： 是否存在 $T$ 使得 $M(T) = v$ 是可判定的。



可以进一步改进字码为：当Turing机集合 $U=\{T_1, \cdots, T_u \}$ $T_1$ 和 $T_2$ 实际上为相同的Turing机，只是规则的编码顺序不同而导致各Turing机计算出来的字码不同时，使用 $\min\{ M(T) |T \in U\}$ 作为 $U$ 中所有Turing机共同的字码。据此可以得到Turing机集合到自然数集合的一对一映射。



# 参考文献

[1] 黄文奇, 许如初. 近世计算理论导引: NP 难度问题的背景, 前景及其求解算法研究[M]. 科学出版社, 2004: 1-10.

