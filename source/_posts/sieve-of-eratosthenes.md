---
title: Sieve of Eratosthenes 埃拉托色尼筛选法
tags: []
categories: ["algorithm"]
reward: true
copyright: true
date: 2019-10-06 01:25:24
thumbnail:
---







埃拉托色尼筛选法 (sieve of Eratosthenes) 由古希腊人发明，它可以给出小于 n 的所有质数。

<!--more-->



## 基本思想

该算法首先初始化一个 2 到 $n$ 的连续整数序列，作为候选质数。接下来从2开始往后循环，每个循环，它会将 2 到 $n$ 范围内该数的所有倍数从序列中剔除。若循环到一个已经被去除的数，则跳过(因为这个数字在之前的循环中已经被剔除，这意味着它的倍数也必定被剔除了，没有必要再剔除一遍)。实际上只需要从 2 循环到 $\lfloor \sqrt{n} \rfloor$ 即可，因为大于 $\lfloor \sqrt{n} \rfloor$ 的整数在 2 到 $n$ 的范围不存在倍数。



## 伪代码



$$
\begin{array}{l}
\hline
{\textbf{Algorithm   Sieve of Eratosthenes}  } \\
\hline
{\textbf{Input:} \quad \text{一个整数} \ n \quad (n>1) } \\
{\textbf{Output:} \quad \text{包含所有小于等于} \ n \ \text{的质数的数组} \ L } \\

{\qquad \textbf{for} \enspace i \leftarrow 2 \enspace \textbf{to} \enspace n \enspace \textbf{do}}\\
{\qquad \qquad A[i] \leftarrow i }\\
{\qquad \textbf{end for} } \\
{\qquad \textbf{for} \enspace i \leftarrow 2 \enspace \textbf{to} \enspace \lfloor \sqrt{n} \rfloor \enspace \textbf{do}  } \\
{\qquad \qquad \textbf{if} \enspace A[i] \ne 0} \enspace \textbf{do} \\
{\qquad \qquad \qquad j \leftarrow i * i} \\
{\qquad \qquad \qquad \textbf{while} \enspace j \le n \enspace \textbf{do}} \\
{\qquad \qquad \qquad \qquad A[j] \leftarrow 0} \\
{\qquad \qquad \qquad \qquad j \leftarrow j + i} \\
{\qquad \qquad \qquad \textbf{end while}} \\
{\qquad \qquad \textbf{end if}} \\
{\qquad \textbf{end for}} \\
{\qquad j \leftarrow 0 } \\
{\qquad \textbf{for} \enspace i \leftarrow 2 \enspace \textbf{to} \enspace n \enspace \textbf{do}}\\
{\qquad \qquad \textbf{if} \enspace A[i] \ne 0 \enspace \textbf{do}}\\
{\qquad \qquad \qquad L[j] \leftarrow i} \\
{\qquad \qquad \qquad j \leftarrow j + 1} \\
{\qquad \qquad \textbf{end if}}\\
{\qquad \textbf{end for} } \\
{\qquad \textbf{return} \enspace L } \\
\hline
\end{array}
$$


## 时间复杂度

$$
O (n \log \log n)
$$

