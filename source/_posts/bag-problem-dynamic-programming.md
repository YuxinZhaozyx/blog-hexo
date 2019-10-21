---
title: 背包问题动态规划求解
tags: ["dynamic-programming", "bag-problem"]
categories: ["algorithm"]
reward: true
copyright: true
date: 2019-10-21 20:17:02
thumbnail:
---



本文介绍通过动态规划求解背包问题。

<!--more-->

# 问题

给定 $n$ 个重量为 $w_1, \cdots, w_n$ ，价值为 $v_1, \cdots, v_n$ 的物品和一个承重量为 $W$ 的背包，求这些物品中最有价值的一个子集，并且要能够装进背包里。

# 算法思想

设 $F(i,j)$ 为物品重量为 $w_1, \cdots, w_i$ ，价值为 $v_1, \cdots, v_i$ 的物品和一个承重量为 $j$ 的背包的背包问题的最优解的总价值。也就是说 $F(i,j)$ 是能够放进承重量为 $j$  的背包的前 $i$ 个物品中最有价值子集的总价值。

**推导式：**

$$
F(i, j) = 
\begin{cases}
\max \{F(i-1, j), v_i + F(i-1, j-w_i) \}, &j-w_i \ge 0 \\
F(i-1, j), &j-w_i \lt 0
\end{cases}
$$

**边界条件：**

$$
F(i, 0) = 0 , \forall i \\
F(0, j) = 0 , \forall j
$$

# 代码实现

## C++实现

```c++
#include <iostream>

int Max(int a, int b) {
    return a > b ? a : b;
}

/* Dynamic programming method to solve bag problem
 *
 * Args:
 *      dst (int*): pointer to the output array of size (n+1) x (maxBagWeight+1), index of (i, j) means the maximum value if 
 *                  the bag max weight equals to j and only consider the 1 to i items.
 *      numItem (int): the number of items.
 *      maxBagWeight (int): the max weight of bag.
 *      value (int*): the value of items. array of shape n
 *      weight (int*): the weight of items. array of shape maxBagWeight
 * 
 * Returns:
 *      int: the max value the bag can store.
 */
int BagDP(int* dst, int numItem, int maxBagWeight, int* value, int* weight) {
    // initialize the bounder
    for (int i = 0; i <= numItem; i++) dst[i * (maxBagWeight + 1)] = 0;
    for (int j = 0; j <= maxBagWeight; j++) dst[j] = 0;
    
    // calcualte the max weight
    for (int i = 1; i <= numItem; i++) {
        for (int j = 1; j <= maxBagWeight; j++) {
            dst[i * (maxBagWeight + 1) + j] = j - weight[i - 1] >= 0 ? Max(dst[(i - 1) * (maxBagWeight + 1) + j], value[i - 1] + dst[(i - 1) * (maxBagWeight + 1) + (j - weight[i - 1])]) : dst[(i - 1) * (maxBagWeight + 1) + j];
        }
    }

    return dst[numItem * (maxBagWeight + 1) + maxBagWeight];
}

void Test() {
    const int numItem = 4;
    const int maxBagWeight = 5;

    int value[numItem] = {12, 10, 20, 15};
    int weight[numItem] = {2, 1, 3, 2};

    int dst[numItem + 1][maxBagWeight + 1];

    int maxValue = BagDP(&dst[0][0], numItem, maxBagWeight, value, weight);
    
    std::cout << "max value : " << maxValue << std::endl;
    for (int i = 0; i <= numItem; i++) {
        for (int j = 0; j <= maxBagWeight; j++) {
            std::cout << dst[i][j] << " ";
        }
        std::cout << std::endl;
    } 
}

int main() {
    Test();
    return 0;
}
```

