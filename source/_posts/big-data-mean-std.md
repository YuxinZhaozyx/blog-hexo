---
title: 大数据量下均值和方差的增量计算
tags: ["big-data", "math"]
categories: ["algorithm"]
reward: true
copyright: true
date: 2020-05-13 22:25:28
thumbnail:
---



本文介绍如何在大数据量下增量计算均值和方差。

<!--more-->

假设我们有两组数据，一组历史数据：$h_1, h_2, \dots, h_M$，一组增量数据：$a_1, a_2, \dots, a_N$。

历史数据均值：

$$
\bar{H} = \frac{1}{M} \sum_{i=1}^M h_i
$$

历史数据方差：

$$
\sigma_H^2 = \frac{1}{M} \sum_{i=1}^M (h_i -\bar{H})^2
$$

增量数据均值：

$$
\bar{A} = \frac{1}{N} \sum_{j=1}^N a_j
$$

增量数据方差：

$$
\sigma_A^2 = \frac{1}{N} \sum_{j=1}^N (a_j -\bar{A})^2
$$

我们希望求得历史数据和增量数据的总均值 $\bar{X}$ 和方差 $\sigma^2$。

$$
\bar{X} = \frac{1}{M+N} \left( \sum_{i=1}^M h_i + \sum_{j=1}^N a_j \right) = \frac{M \bar{H} + N \bar{A}}{M + N}
$$

$$
\begin{align}
\sigma^2 &= \frac{1}{M+N} \left[ \sum_{i=1}^M (h_i -\bar{X})^2 + \sum_{j=1}^N (a_j - \bar{X})^2 \right] \\
&= \frac{1}{M+N} \left[ \sum_{i=1}^M \left( (h_i - \bar{H}) -(\bar{X} - \bar{H}) \right)^2 + \sum_{j=1}^N \left( (a_j - \bar{A}) - (\bar{X} -\bar{A}) \right)^2 \right] \\
&= \frac{1}{M+N} \left[ \sum_{i=1}^M \left( (h_i - \bar{H})^2 -2(h_i - \bar{H})(\bar{X} - \bar{H}) + (\bar{X} - \bar{H})^2 \right) \\ + \sum_{j=1}^N \left( (a_j - \bar{A})^2 - 2(a_j - \bar{A})(\bar{X} -\bar{A})  + (\bar{X} -\bar{A})^2 \right) \right] \\
&= \frac{1}{M+N} \left[ M \sigma_H^2 + M(\bar{X}-\bar{H})^2-2(\bar{X} - \bar{H})(\sum_{i=1}^M h_i - M \bar{H}) \\ + N \sigma_A^2 + N(\bar{X} - \bar{A})^2 - 2(\bar{X} - \bar{A})(\sum_{j=1}^N a_j - N \bar{A}) \right] \\
&= \frac{1}{M+N} \left[ M \sigma_H^2 + M(\bar{X} - \bar{H})^2 + N \sigma_A^2 +N(\bar{X} - \bar{A})^2 \right] \\
&= \frac{M \left[ \sigma_H^2 + (\bar{X} - \bar{H})^2 \right]  + N \left[ \sigma_A^2 + (\bar{X} - \bar{A})^2 \right]}{M+N}
\end{align}
$$



