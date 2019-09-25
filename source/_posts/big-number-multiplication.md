---
title: "大整数乘法"
tags: ["math"]
categories: ["algorithm"]
reward: true
copyright: true
date: 2019-09-23 19:57:29
thumbnail:
---



通常两个n位十进制整数相乘 (如果两个数位数不同则在位数少的数字前填0至位数相同)，第一个数中的n个数字都分别要与第二个数中的n个数字相乘，这样一共就要做 $n^2$ 次位乘，因此时间复杂度为 $\Theta(n^2)$。本文介绍如何优化乘法的时间复杂度。

<!--more-->



以两个整数23和14相乘为例，这两个数字可以表示为：


$$
23 = 2 \times 10^1 + 3 \times 10^0 \\
14 = 1 \times 10^1 + 4 \times 10^0
$$


两者相乘为


$$
\begin{align}
23 \times 14 & = (2 \times 10^1 + 3 \times 10^0) \times (1 \times 10^1 + 4 \times 10^0) \\
&= (2 \times 1) \times 10^2 + (2 \times 4 + 3 \times 1) \times 10^1 + (3 \times 4) \times 10^0 \\
\end{align}
$$


这个算法跟笔算算法相同，都会进行4次位乘 ($2 \times 1$,  $2 \times 4$,  $3 \times 1$,  $3 \times 4$)，但我们可以通过重复使用 $2 \times 1$ 和 $3 \times 4$ 的计算结果，通过将中间项 $(2 \times 4 + 3 \times 1)$ 化简减少位乘的次数:


$$
2 \times 4 + 3 \times 1 = (2 + 3) \times (1 + 4) - (2 \times 1) - (3 \times 4)
$$


**将它一般化一点：**

对于任何两位数 $a = a_1 a_0$ 和 $b = b_1 b_0$ 来说，它们的积 $c$ 可以用下列公式计算：


$$
\begin{align}
c_2 &= a_1 \times b_1 \\
c_0 &= a_0 \times b_0 \\
c_1 &= (a_1 + a_0) \times (b_1 + b_0) - (c_2 + c_0) \\
c &= a \times b = c_2 \times 10^2 + c_1 \times 10^1 + c_0
\end{align}
$$


**运用递归将其更一般化：**

对任意n位数 $a$ 和 $b$，采用分治法将 n 位整数拆成两个 n/2 位整数，即 $ a = a_1 \times 10^{n/2} + a_0$，$ b = b_1 \times 10^{n/2} + b_0$ 。使用与前面两位数的相同方法，可得：


$$
\begin{align}
c_2 &= a_1 \times b_1 \\
c_0 &= a_0 \times b_0 \\
c_1 &= (a_1 + a_0) \times (b_1 + b_0) - (c_2 + c_0) \\
c &= a \times b = c_2 \times 10^n + c_1 \times 10^{n/2} + c_0
\end{align}
$$


计算时 $a_1 \times b_1$ 等 n/2 位数乘法同n位乘法，递归计算下去即可。

**时间复杂度计算：**

因为n位数的乘法需要对n/2位数做3次乘法，乘法次数 $M(n)$ 的递推式：


$$
M(n) =
\begin{cases}
3 M(n/2), &n>1 \\
1, &n=1
\end{cases}
$$


当 $n=2^k$ 时，


$$
M(2^k) = 3 M(2^{k-1}) = 3 \left[ 3 M(2^{k-2}) \right] = \cdots = 3^k
$$


因为 $k=\log_2 n$ ，


$$
M(n) = 3^{\log_2 n} = n^{log_2 3} \approx n^{1.585}
$$


以上我们只考虑了乘法次数，接下来考虑加法次数 $A(n)$：

每次需要4次加运算和1次减运算，每个运算都有n次位运算，因此：


$$
A(n) = 
\begin{cases}
3 A(n/2) + 5n, &n>1 \\
1, &n=1
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
A(n) \in \Theta(n^{\log_2 3})
$$


于是算法的总时间复杂度为 $\left( M(n) + A(n) \right) \in \Theta(n^{log_2 3})$.