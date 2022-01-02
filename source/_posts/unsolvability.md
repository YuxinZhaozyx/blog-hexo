---
title: 不可计算性(不可解性)
tags: ["algorithm", "math"]
categories: ["algorithm"]
reward: true
copyright: true
date: 2022-1-1 14:20:41
thumbnail:
---



本文是《近世计算理论导引：NP难度问题的背景、前景及其求解方法研究》的第二章《不可计算性》的笔记。

<!--more-->

# 胜弈机的不存在性

胜弈机和“上帝是万能的”问题类似，即存在一个计算机系统能够在国际象棋中击败任何对手。

如果有两个胜弈机互相博弈，结局为一负一胜或者平局，出现至少一个胜弈机与胜弈机定义矛盾的情况，因此胜弈机不存在。



# 不可计算函数的存在性

自然数集合 $\mathbb{N}$ 为可数无穷集合，记有可数个元素 $N_0$，即 $\mathbb{N} = \{0, N_0 - 1\}$, $N_0 \to +\infty$。

此处讨论的函数为映射 $f: \mathbb{N} \mapsto \mathbb{N}$ ，是一个全函数。



**定理1：不同程序的总个数不超过 $\mathbb{N}$ 的元素个数 $N_o$**。

+ 程序是一个有穷字符串，将Turing机的字符表 $A=\{a_1,\cdots,a_v\}$ 的 $A^*$ 中的元素按照以下顺序排列：

  $$
  \phi;a_1,\cdots,a_v;a_1a_1,\cdots,a_va_v;\cdots
  $$
  
  即可将 $A^*$ 中的有穷字符串以自然数进行编号，建立起从程序到 $\mathbb{N}$的一一映射，即穷字符串共有 $N_0$ 个。其中只有部分字符串为合法的程序，因此程序的总个数 $\le N_0$。



**定理2：数论函数有 $N_0^{N_0}$ 个。**

+ 对于 $x \in \mathbb{N}$，$f(x)$ 有 $N_0$ 个可能输出，而 $x$ 共有 $N_0$ 个，因此 $f$ 总共有 $N_0^{N_0}$ 个可能性。



**定理3：可计算函数总共恰好有 $N_0$ 个互不相同。**

+ 由定理1可知不同程序的个数最多为 $N_0$ 个，而每个程序至多计算一个函数，因此有程序计算的函数至多有 $N_0$ 个；假定 $f$ 为可计算函数，则 $c f$, $c \in \mathbb{N}$ 这 $N_0$ 个函数也为可计算函数；综上可计算函数恰好有 $N_0$ 个互不相同。



**定理4：$2^{N_0} > N_0$。**



**定理5：存在着不可计算的函数。**

+ $N_0^{N_0} \ge 2^{N_0} \gt N_0$ 说明数论函数的数量 $N_0^{N_0}$ 大于可计算的函数的数量 $N_0$，因此肯定有某些数论函数无法用程序计算。 



# 停机问题的不可解性

设 $T_1, T_2, \cdots$ 为全体不同的合法程序。

对任一程序 $T_i$，当给出输入之后启动，它有停机和不停机两种结局。停机后则会给出输出。

对于程序的输入和输出我们都可以用自然数进行编码，同时记停机为 $\downarrow$，不停机为 $\uparrow$。

任一程序 $T_i$ 都相当于一个函数 $\varphi_i$ ，其定义域为 $\{0,1,2,\cdots\}$，值域为 $\{\uparrow, 0, 1,2, \cdots\}$。



**定义1：递归可枚举集(r.e. 集)**

+ 说 $\mathbb{N}$ 的某个子集 $A$ 为递归可枚举集是指它可以表示为某个 $\varphi_i$ 的定义域，即 $(\exists i)[A=\{x|\varphi_i (x) \downarrow\}]$ 。



**定义2：可计算集**

+ 说 $\mathbb{N}$ 的某个子集 $A$ 为可计算集是指存在一个函数，对于任意给定的自然数 $x$， $x \in ? A$ 的问题是可以由该程序确切判断的。



**引理1：可计算集为 r.e. 集。**

+ 设 $A$ 为可计算集，定义可计算函数 $f$:

  $$
  f(x) = \begin{cases} 1, &x \in A \\ \uparrow, &x \notin A \end{cases}
  $$
  
  于是 $A=\{x| f(x) \downarrow \}$ 。



**引理2：可计算集的余集也是可计算集。**

+ 设 $A$ 为可计算集，则可判断 $x \in ? A$。若 $x \in A$  则 $x \notin \bar A$， 若 $x \notin A$ 则 $x \in \bar A$。因此可判断 $x \in ? \bar A$，即 $\bar A$ 是可计算集。



**引理3：存在集 $\Theta$ 不是 r.e. 集。**

+ 记 $w_i = \{x|\varphi_i (x) \downarrow\}$，$w_i$ 是 r.e. 集。
+ 构造 $\Theta = \{ i | i \notin w_i \} = \{ i | \varphi_i (i) \uparrow \}$ ，由于 $\forall i:\Theta \neq w_i$，$\Theta$ 不是 r.e. 集。
+ 由引理1知 $\Theta$ 不是可计算集。
+ $\Theta$ 的余集记作 $K$，即 $K=\bar \Theta=\{i|\varphi_i(i) \downarrow \}$。



**定理1：$K$ 不是可计算集。**

+ 由引理3知 $\Theta$ 不是可计算集。$K = \bar \Theta$，于是由引理2可知 $K$ 不是可计算集。



# Turing机停机问题之Turing机不可解性

**定理1：任给自然数 $x$，问 $\varphi_x(x)$ 是否收敛，这个问题是不可能由某个Turing机统一解决的。**

+ 设该问题可以由某一个Turing机 $T_{\theta}$ 解决，即

  $$
  \varphi_\theta(x) = \begin{cases} 1, &\varphi_x(x) \downarrow \\ \uparrow, &\varphi_x(x) \uparrow \end{cases}
  $$
  
  若该问题可解，则“任给自然数 $x$，问 $\varphi_x(x)$ 是否不收敛”个问题同样可解，即存在一个Turing机 $T_v$ 使得
  
  $$
  \varphi_v(x) = \begin{cases} \uparrow, &\varphi_x(x) \downarrow \\ 1, &\varphi_x(x) \uparrow \end{cases}
  $$
  
  因此 $\varphi_v(v) \uparrow \Leftrightarrow \varphi_v(v) \downarrow$，矛盾。





# 参考文献

[1] 黄文奇, 许如初. 近世计算理论导引: NP 难度问题的背景, 前景及其求解算法研究[M]. 科学出版社, 2004: 11-17.

