---
title: "Strassen矩阵乘法"
tags: ["math"]
categories: ["algorithm"]
reward: true
copyright: true
date: 2019-09-23 22:01:20
thumbnail:
---



本文将介绍Strassen矩阵乘法，利用分治思想减少矩阵乘法所需要的计算量。

<!--more-->



假定我们要求两个 $n \times n$ 矩阵 $A$ 和 $B$ 的矩阵乘法结果 $C$, 即 $C = A \times B$。

假定 $n = 2^k, k \in \mathbb{Z}^+$, 我们可以将 $A$, $B$, $C$ 划分为4个 $(n/2) \times (n/2)$ 的子矩阵。


$$
\begin{align}
\begin{bmatrix}
C_{00} & C_{01} \\
C_{10} & C_{11} 
\end{bmatrix}
&= 
\begin{bmatrix}
A_{00} & A_{01} \\
A_{10} & A_{11} 
\end{bmatrix}
\times
\begin{bmatrix}
B_{00} & B_{01} \\
B_{10} & B_{11} 
\end{bmatrix}
\\
&=
\begin{bmatrix}
M_1 + M_4 -M_5 +M_7 & M_3 + M_5 \\
M_2 + M_4 & M_1 + M_3 - M_2 + M_6
\end{bmatrix}
\end{align}
$$



$$
\begin{align}
M_1 &= (A_{00} + A_{11}) \times (B_{00}+B_{11}) \\
M_2 &= (A_{10} + A_{11}) \times B_{00} \\
M_3 &= A_{00} \times (B_{01} - B_{11}) \\
M_4 &= A_{11} \times (B_{10} - B_{00}) \\
M_5 &= (A_{00} + A_{01}) \times B_{11}) \\
M_6 &= (A_{10} - A_{00}) \times (B_{00} + B_{01}) \\
M_7 &= (A_{01} - A_{11}) \times (B_{10} + B_{11})
\end{align}
$$


**时间复杂度分析：**

乘法次数 $M(n)$ :


$$
M(n) = 
\begin{cases}
7 M(n/2), &n>1 \\
1, &n=1
\end{cases}
$$


因为 $n=2^k$,


$$
M(2^k) = 7 M(2^{k-1}) = 7 \left[ 7 M(2^{k-2}) \right] = \cdots = 7^k
$$


因为 $k= \log_2 n$,


$$
M(n) = 7^{\log_2 n} = n^{\log_2 7} \approx n^{2.807}
$$


减少的乘法次数是以额外的加减法次数为代价的，因此我们需要计算加减法次数 $A(n)$,


$$
A(n) = 
\begin{cases}
7 A(n/2) + 18 (n/2)^2 , &n>1 \\
0, &n=1
\end{cases}
$$


根据主定理：


$$
T(n) = a T(n/b) + f(n) \\
f(n) \in \Theta(n^d) \\
T(n) \in 
\begin{cases}
\Theta(n^d), &a \lt b^d \\
\Theta(n^d \log n), &a=b^d \\
\Theta(n^{\log_b a}), &a>b^d 
\end{cases}
$$


可以推出


$$
A(n) \in \Theta(n^{\log_2 7})
$$



于是算法的总时间复杂度为 $\left( M(n) + A(n) \right) \in \Theta(n^{log_2 7})$.