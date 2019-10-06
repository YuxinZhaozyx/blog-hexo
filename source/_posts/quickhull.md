---
title: Quickhull 快包算法解决凸包问题
tags: ["convex-hull"]
categories: ["algorithm"]
reward: true
copyright: true
date: 2019-10-06 12:10:37
thumbnail: /static/image/quickhull.png
---





**凸包问题 (convex-hull problem)** 是为一个有 n 个点的集合构造凸包的问题, 或者更直观的解释：求能够完全包含平面上 n 个给定点的凸多边形。



<!--more-->

## 定义

**凸集合定义：**  对于平面上的一个点集合 (有限或者无限)， 如果以集合中任意两点 $p$ 和 $q$ 为端点的线段都属于该集合，则这个集合是凸集合。 

**凸包(convex hull)定义：** 包含点集合 $S$ 的最小凸集合 (“最小”指的是 $S$的凸包一定是所有包含 $S$ 的凸集合的子集)。

+ 直观地讲，对于平面上 $n$ 个点的集合，它的凸包就是包含所有这些点的最小凸多边形。

**凸包的极点(extreme point)定义：**  对于任何以集合中的点为端点的线段来说，它们不可能是这种线段的中点。

+ 直观地讲，凸包形成的凸多边形的各个顶点就是凸包的极点。 



![convex-hull](quickhull/%E6%9C%AA%E5%91%BD%E5%90%8D%E8%A1%A8%E5%8D%95%20(11).png)



## 算法

假设集合 $S$ 是平面上 $n>1$ 个点 $p_1(x_1, y_1), \cdots, p_n(x_n, y_n)$ 构成的，假设这些点是按照x轴坐标升序排列的，如果x坐标相同则按照y坐标排列。由此，最左边的点必定是 $p_1$, 最右边的点必定是 $p_n$，这两个点必定是凸包的两个极点。

直线 $\overrightarrow{p_1 p_n}$ 将集合分为两个子集：上集合 $S_1$,  下集合 $S_2$ 。两个子集对应的凸包我们称之为 上包 (upper hull) 和 下包 (lower hull)。

以下解释构建上包的原理，下包同理：

+ 如果 $S_1$ 为空，上包就是以 $p_1$ 和 $p_n$ 为端点的线段。
+ 如果 $S_1$ 不为空，找到 $S_1$ 中的距离直线 $\overrightarrow{p_1 p_n}$ 最远的点作为上包的顶点 $p_{max}$ 。如果距离最远的点有多个则选择能使 $\angle p_{max} p_1 p_n$ 最大的点。
+ 接着求 $\overrightarrow{p_1 p_{max}}$ 和 $\overrightarrow{p_{max} p_n}$  的上包，递归执行。每次求上包都能找到 $S$ 上包的一个顶点，最终找到 $S$ 上包的所有顶点。
+ 下包进行同样的操作，最终即可获得 $S$ 的凸包的所有顶点。凸包构建完成。



![quick-hull](quickhull/%E6%9C%AA%E5%91%BD%E5%90%8D%E8%A1%A8%E5%8D%95%20(12).png)

### Tips

运用行列式


$$
\begin{vmatrix} 
x_1 & y_1 & 1 \\
x_2 & y_2 & 1 \\
x_3 & y_3 & 1 
\end{vmatrix}
= x_1 y_2 + x_3 y_1 + x_2 y_3 - x_3 y_2 - x_2 y_1 - x_1 y_3
$$


可以在固定时间内判断 点 $p_3=(x_3, y_3)$ 是在直线 $\overrightarrow{p_1 p_2}$ 的左侧还是右侧，且可得到该点到直线的距离。当且仅当 $p_3$ 位于直线 $\overrightarrow{p_1 p_2}$ 的左侧时，行列式值为正。点 $p_3$ 到直线 $\overrightarrow{p_1 p_2}$ 的距离为行列式的绝对值。

## 时间复杂度

+ **最佳情况**：$\Theta (n)$
+ **最坏情况**：$\Theta (n^2)$



## 实现

### C++实现



```c++
#include <iostream>
#include <algorithm>
#include <vector>

struct Vertex {
    double x;
    double y;
};

/*  Compare two vertices, determine whether vertex a < b */
bool CompareVertex(Vertex a, Vertex b) {
    if (a.x == b.x) {
        return a.y < b.y;
    }
    else {
        return a.x < b.x;
    }
}

/* Find the upper or lower hull */ 
std::vector<Vertex> __findHull(std::vector<Vertex> originSet, Vertex p1, Vertex p2, bool findUpper){
    std::vector<Vertex> upperSet;

    Vertex p_max;
    double maxDistance = -1.0;
    for (auto iter = originSet.begin(); iter != originSet.end(); iter++) {
        double distance = p1.x * p2.y + (iter->x) * p1.y + p2.x * (iter->y) - (iter->x) * p2.y - p2.x * p1.y - p1.x * (iter->y);
        if ((findUpper && distance > 0.0) || (!findUpper && distance < 0.0)) {
            upperSet.push_back(*iter);
            distance = std::abs(distance);
            if (distance > maxDistance) {
                p_max = (*iter);
                maxDistance = distance;
            }
        }
    }
    
    if (maxDistance > 0.0) {
        std::vector<Vertex> SubHull1 = __findHull(upperSet, p1, p_max, findUpper);
        std::vector<Vertex> SubHull2 = __findHull(upperSet, p_max, p2, findUpper);
        SubHull1.insert(SubHull1.begin(), SubHull2.begin(), SubHull2.end());
        SubHull1.push_back(p_max);
        return SubHull1;
    }
    else {
        return std::vector<Vertex>(); // empty
    }

}

/* Algorithm: Qucik Hull
 * 
 * Args:
 *     originSet (std::vector<Vertex>): set of n vertices 
 * 
 * Outputs:
 *     std::vector<Vertex>: convex hull
 */
std::vector<Vertex> QuickHull(std::vector<Vertex> originSet){
    // sort all vertice
    std::sort(originSet.begin(), originSet.end(), CompareVertex);

    // find upper and lower hulls
    std::vector<Vertex> SubHull1 = __findHull(originSet, originSet.front(), originSet.back(), true);
    std::vector<Vertex> SubHull2 = __findHull(originSet, originSet.front(), originSet.back(), false);

    // combine upper and lower hulls
    SubHull1.insert(SubHull1.begin(), SubHull2.begin(), SubHull2.end());
    SubHull1.push_back(originSet.front());
    SubHull1.push_back(originSet.back());

    return SubHull1;
}

void test() {
    std::vector<Vertex> originSet = {{1.0, 1.0}, {2.0, 2.0}, {3.0, 1.0}, {3.0, 3.0} , {1.0, 3.0}};
    
    originSet = QuickHull(originSet);

    for (auto vertex: originSet){
        std::cout << vertex.x << ' ' << vertex.y << std::endl; 
    }
}

int main() {
    test();
    return 0;
}
```

