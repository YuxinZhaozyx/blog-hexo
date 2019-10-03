---

title: Matrix Calculus 矩阵微积分
tags: ["math"]
categories: ["machine-learning"]
reward: true
copyright: true
date: 2019-10-02 21:50:33
thumbnail:
---





本文主要介绍矩阵如何求导。

我们可用[网站](http://www.matrixcalculus.org/)帮助我们对矩阵进行求导，但更重要的是我们要掌握矩阵求导的基本理论。

<!--more-->





# 标量对矩阵求导

## 定义

从定义来考虑，标量 $f$ 对矩阵 $X$ 的导数定义为 $f$ 对 $X$ 逐元素求导排成与 $X$ 尺寸相同的矩阵：

$$
\frac{\partial f}{\partial X} = \left[ \frac{\partial f}{\partial X_{ij}} \right]
$$

但这个定义并不好计算，因此我们试图寻找一个从整体出发、不需要在求导时拆开矩阵的算法。、

**一元微积分中导数与微分的关系：**

$$
\mathrm{d}x = f'(x) \mathrm{d}x
$$

**多元微积分中梯度(标量对向量的导数)与微分的关系：**


$$
\mathrm{d} f = \sum_{i=1}^n \frac{\partial f}{\partial x_i} \mathrm{d} x_i = \left( \frac{\partial f}{\partial \boldsymbol{x}} \right)^T \mathrm{d} \boldsymbol{x}
$$

受此启发，我们**将矩阵导数与微分建立联系：**


$$
\mathrm{d} f = \sum_{i=1}^m \sum_{j=1}^n \frac{\partial f}{\partial X_{ij}} \mathrm{d} X_{ij} = \mathrm{tr} \left( \left(\frac{\partial f}{\partial X}\right)^T \mathrm{d} X \right)
$$

+ 其中 $\mathrm{tr}(\cdot)$ 代表迹(trace)即方阵对角线元素之和，满足性质：
  + 对于尺寸相同的矩阵 $A$, $B$, $\mathrm{tr}(A^T B)$ 是矩阵 $A$, $B$ 的内积：
  
    $$
    \mathrm{tr}(A^T B) = \sum_{i,j} A_{ij} B_{ij}
    $$



## 矩阵微分的运算法则

1. **加减法：** $\mathrm{d} (X \pm Y) = \mathrm{d} X \pm \mathrm{d} Y$

2. **矩阵乘法：** $\mathrm{d}(XY) = (\mathrm{d} X) Y + X\mathrm{d}Y$

3. **转置：** $\mathrm{d}(X^T) = (\mathrm{d} X)^T$

4. **逆：**   $\mathrm{d} (X^{-1}) = -X^{-1}(\mathrm{d}X)X^{-1}$ 

5. **迹：** $\mathrm{d} \left( \mathrm{tr}(X) \right) = \mathrm{tr} (\mathrm{d} X)$

6. **行列式：** $\mathrm{d} | X | = \mathrm{tr} \left( X^* \mathrm{d} X \right)$， 其中 $X^*$ 表示 $X$ 的伴随矩阵，在 $X$ 可逆时又可写作 $\mathrm{d} | X | = \mathrm{tr} \left( X^{-1} \mathrm{d} X \right)$

7. **Kronecker乘积：**   $\mathrm{d} (X \otimes Y) = \mathrm{d} X \otimes Y + X \otimes \mathrm{d} Y $

8. **逐元素乘法：** $\mathrm{d} (X \odot Y) = \mathrm{d} X \odot Y + X \odot \mathrm{d} Y$，$\odot$ 表示尺寸相同的矩阵$X$, $Y$逐元素相乘

9. **逐元素函数：** $\mathrm{d} \sigma(X) = \sigma'(X) \odot \mathrm{d} X$，其中，$\sigma(X) = \left[ \sigma(X_{ij}) \right]$ , $\sigma'(X) = \left[ \sigma'(X_{ij}) \right]$



## 迹的运算法则

1. **标量的迹：** $a = \mathrm{tr}(a)$

2. **转置：** $\mathrm{tr} \left( A^T \right) = \mathrm{tr}(A)$

3. **线性：** $\mathrm{tr}(A \pm B) = \mathrm{tr}(A) \pm \mathrm{tr}(B)$

4. **矩阵乘法交换：** $\mathrm{tr}(AB) = \mathrm{tr}(BA)$

5. **矩阵乘法/逐元素乘法交换：** $\mathrm{tr} \left( A^T (B \odot C) \right) = \mathrm{tr} \left( (A \odot B)^T C \right)$, 其中 $A$, $B$, $C$ 尺寸相同，等号两侧都等于 $\sum_{ij} A_{ij} B_{ij} C_{ij}$







## 求解方法

**若标量函数 $f$ 是矩阵 $X$ 经加减乘法、逆、行列式、逐元素函数等运算构成，则使用相应的运算法则对 $f$ 求微分，再使用迹技巧给 $df$ 套上迹并使其他项交换至 $\mathrm{d} X$ 左侧，对照导数与微元的联系 $\mathrm{d} f = \mathrm{tr} \left( \left(\frac{\partial f}{\partial X}\right)^T \mathrm{d} X \right)$， 即可求得导数。**



**复合：** 假设已求得 $\frac{\partial f}{\partial Y}$， 而 $Y$ 是 $X$ 的函数，求 $\frac{\partial f}{\partial X}$。 此时，由链式法则 $\frac{\partial f}{\partial X} = \frac{\partial f}{\partial Y} \frac{\partial Y}{\partial X}$似乎可求解，但问题是 $\frac{\partial Y}{\partial X}$ 我们目前还没有定义，因此此处不能用链式法则，我们需要找其他方法。我们从微分入手建立复合法则，先写出 $\mathrm{d} f = \mathrm{tr} \left( \left(\frac{\partial f}{\partial Y} \right)^T \mathrm{d} Y \right)$ ，再将 $\mathrm{d} Y$ 用 $\mathrm{d} X$ 表示出来代入，并用迹的运算法则将其他项交换到 $\mathrm{d} X$ 左侧， 即可得到 $\frac{\partial f}{\partial X}$。

**例子：**

以 $Y = AXB$ 为例， 此时

由于 $A$, $B$ 为常量，$\mathrm{d} A = 0$, $\mathrm{d} B = 0$

$$
\mathrm{d} Y = (\mathrm{d} A) X B + A(\mathrm{d}X)B + AX(\mathrm{d}B) = A(\mathrm{d}X)B
$$

$$
\begin{align}
\mathrm{d} f &= \mathrm{tr} \left( \left(\frac{\partial f}{\partial Y} \right)^T \mathrm{d} Y \right) \\
&= \mathrm{tr} \left( \left(\frac{\partial f}{\partial Y} \right)^T A (\mathrm{d} X) B \right) \\
&=  \mathrm{tr} \left( B\left(\frac{\partial f}{\partial Y} \right)^T A \mathrm{d} X \right) \\
&= \mathrm{tr} \left( \left(A^T \frac{\partial f}{\partial Y} B^T \right)^T  \mathrm{d} X \right)
\end{align}
$$

可得 $\frac{\partial f}{\partial X} = A^T \frac{\partial f}{\partial Y} B^T$ 。



## 案例

**例1：**   $f = A^T X B$,   $f \in \mathbb{R}$,  $A \in \mathbb{R}^{m \times 1}$,  $X \in \mathbb{R}^{m \times n}$,  $B \in \mathbb{R}^{n \times 1}$,  求 $\frac{\partial f}{\partial X}$。

**解：**

$$
\begin{align}
\mathrm{d} f &= (\mathrm{d} (A^T)) X B  + A^T (\mathrm{d} X) B + A^T X (\mathrm{d} B) \\
& = A^T (\mathrm{d} X) B 

\\\\

\mathrm{d} f &= \mathrm{tr} \left( \mathrm{d} f \right) \\
&= \mathrm{tr} \left( A^T (\mathrm{d} X) B \right) \\
&= \mathrm{tr} \left( B A^T (\mathrm{d} X) \right) \\
&= \mathrm{tr} \left( (A B^T)^T \mathrm{d} X \right)
\end{align}
$$

由此可得 $\frac{\partial f}{\partial X} = A B^T$。





**例2：**   $f = A^T \exp (X B)$,   $f \in \mathbb{R}$,  $A \in \mathbb{R}^{m \times 1}$,  $X \in \mathbb{R}^{m \times n}$,  $B \in \mathbb{R}^{n \times 1}$,  $\exp(X) = [\exp(X_{ij})]$, 求 $\frac{\partial f}{\partial X}$。

**解：**

$$
\begin{align}
\mathrm{d} f &= A^T \mathrm{d} \left( \exp(XB) \right) \\
&= A^T \left( \exp(XB) \odot \mathrm{d} (X B) \right) \\
&= A^T \left( \exp(XB) \odot ((\mathrm{d}X) B) \right) 

\\
\\

\mathrm{d} f &= \mathrm{tr} \left( \mathrm{d} f \right) \\
&= \mathrm{tr} \left( A^T \left( \exp(XB) \odot ((\mathrm{d}X) B) \right)  \right) \\
&= \mathrm{tr} \left( \left(A \odot \exp(XB) \right)^T (\mathrm{d}X) B   \right) \\
&= \mathrm{tr} \left( B \left(A \odot \exp(XB) \right)^T \mathrm{d}X   \right) \\
&= \mathrm{tr} \left( \left( \left(A \odot \exp(XB) \right) B^T \right)^T \mathrm{d}X  \right) \\
\end{align}
$$

由此可得 $\frac{\partial f}{\partial X} = \left(A \odot \exp(XB) \right) B^T $。



**例3：**  $f = \mathrm{tr} \left( Y^T M Y \right)$,  $Y = \sigma(W X)$,  $f \in \mathbb{R}$,  $W \in \mathbb{R}^{l \times m}$,   $X \in \mathbb{R}^{m \times n}$,    $Y \in \mathbb{R}^{l \times n}$,  $M \in \mathbb{R}^{l \times l}$ 是对称矩阵,  $\sigma(X) = [\sigma(X_{ij})]$, 求 $\frac{\partial f}{\partial X}$。

**解：**


$$
\begin{align}

\mathrm{d} f &=  \mathrm{d} \left( \mathrm{tr} \left( Y^T M Y \right) \right) \\
&= \mathrm{tr} \left( \mathrm{d}(Y^T M Y) \right) \\ 
&= \mathrm{tr} \left( \mathrm{d}(Y^T) M Y + Y^T M \mathrm{d}Y \right) \\
&= \mathrm{tr} \left( \mathrm{d}(Y^T) M Y \right) + \mathrm{tr} \left( Y^T M \mathrm{d}Y \right) \\
&= \mathrm{tr} \left( (\mathrm{d}Y)^T M Y \right) + \mathrm{tr} \left( Y^T M \mathrm{d}Y \right) \\
&= \mathrm{tr} \left( (Y^T M^T \mathrm{d}Y)^T \right) + \mathrm{tr} \left( Y^T M \mathrm{d}Y \right) \\
&= \mathrm{tr} \left( Y^T M^T \mathrm{d}Y \right) + \mathrm{tr} \left( Y^T M \mathrm{d}Y \right) \\
&= \mathrm{tr} \left( Y^T M^T \mathrm{d}Y + Y^T M \mathrm{d}Y \right) \\
&= \mathrm{tr} \left( Y^T (M^T + M) \mathrm{d}Y \right) \\
&= \mathrm{tr} \left( Y^T (2M^T) \mathrm{d}Y \right) \\
&= \mathrm{tr} \left( (2MY)^T \mathrm{d} Y \right) 

\end{align}
$$

可得 $\frac{\partial f}{\partial Y} = 2MY$.

$$
\begin{align}
\mathrm{d} Y &= \mathrm{d} ( \sigma(W X) ) \\
&= \sigma'(WX) \odot \mathrm{d} (WX) \\
&= \sigma'(WX) \odot (W \mathrm{d}X)

\\
\\

\mathrm{d} f &= \mathrm{tr} \left( (\frac{\partial f}{\partial Y})^T \mathrm{d} Y \right) \\
&= \mathrm{tr} \left( (\frac{\partial f}{\partial Y})^T (\sigma'(WX) \odot (W \mathrm{d}X)) \right) \\
&= \mathrm{tr} \left( (\frac{\partial f}{\partial Y} \odot \sigma'(WX) )^T  W \mathrm{d}X \right) \\
&= \mathrm{tr} \left( \left( W^T \left(\frac{\partial f}{\partial Y} \odot \sigma'(WX) \right) \right)^T  \mathrm{d}X \right) \\
&= \mathrm{tr} \left( \left( W^T \left((2MY) \odot \sigma'(WX) \right) \right)^T  \mathrm{d}X \right) \\
&= \mathrm{tr} \left( \left( W^T \left((2M \sigma(WX)) \odot \sigma'(WX) \right) \right)^T  \mathrm{d}X \right) \\

\end{align}
$$

由此可得 $\frac{\partial f}{\partial X} = W^T \left((2M \sigma(WX)) \odot \sigma'(WX) \right) $。



**例4 [线性回归]:**     $l = \| X \boldsymbol{w} - y \|^2$,   $l \in \mathbb{R}$,  $\boldsymbol{y} \in \mathbb{R}^{m \times 1}$,  $X \in \mathbb{R}^{m \times n}$,  $\boldsymbol{w} \in \mathbb{R}^{n \times 1}$,  求 $w$ 的最小二乘估计，即 $\frac{\partial l}{\partial \boldsymbol{w}}$ 的零点。

**解：**

$$
\begin{align}

l &= \| X \boldsymbol{w} - \boldsymbol{y} \|^2 \\
& = (X \boldsymbol{w} - \boldsymbol{y})^T(X \boldsymbol{w} - \boldsymbol{y})

\\
\\

\mathrm{d} l &= \mathrm{d} \left( (X \boldsymbol{w} - \boldsymbol{y})^T(X \boldsymbol{w} - \boldsymbol{y}) \right) \\
&= \mathrm{d} \left( (X \boldsymbol{w} - \boldsymbol{y})^T \right) (X \boldsymbol{w} - \boldsymbol{y}) + (X \boldsymbol{w} - \boldsymbol{y})^T \mathrm{d} (X \boldsymbol{w} - \boldsymbol{y}) \\
&= \left(\mathrm{d}(X \boldsymbol{w} - \boldsymbol{y}) \right)^T (X \boldsymbol{w} - \boldsymbol{y}) + (X \boldsymbol{w} - \boldsymbol{y})^T d(X \boldsymbol{w} - \boldsymbol{y}) \\
&= \left( (X \boldsymbol{w} - \boldsymbol{y})^T \mathrm{d}(X \boldsymbol{w} - \boldsymbol{y})  \right)^T + (X \boldsymbol{w} - \boldsymbol{y})^T \mathrm{d}(X \boldsymbol{w} - \boldsymbol{y}) \\
&= \left( (X \boldsymbol{w} - \boldsymbol{y})^T X\mathrm{d} \boldsymbol{w} \right)^T + (X \boldsymbol{w} - \boldsymbol{y})^T X\mathrm{d} \boldsymbol{w}

\\
\\

\mathrm{d} l &= \mathrm{tr} \left( \mathrm{d} l \right) \\
&= \mathrm{tr} \left( \left( (X \boldsymbol{w} - \boldsymbol{y})^T X\mathrm{d}\boldsymbol{w} \right)^T + (X \boldsymbol{w} - \boldsymbol{y})^T X\mathrm{d}\boldsymbol{w} \right) \\
&= \mathrm{tr} \left( \left( (X \boldsymbol{w} - \boldsymbol{y})^T X\mathrm{d}\boldsymbol{w} \right)^T \right) + \mathrm{tr} \left( (X \boldsymbol{w} - \boldsymbol{y})^T X\mathrm{d}\boldsymbol{w} \right) \\
&= \mathrm{tr} \left( (X \boldsymbol{w} - \boldsymbol{y})^T X\mathrm{d}\boldsymbol{w} \right) + \mathrm{tr} \left( (X \boldsymbol{w} - \boldsymbol{y})^T X\mathrm{d}\boldsymbol{w} \right) \\
&= \mathrm{tr} \left( 2 (X \boldsymbol{w} - \boldsymbol{y})^T X\mathrm{d}\boldsymbol{w} \right) \\
&= \mathrm{tr} \left( (2 X^T (X \boldsymbol{w} - \boldsymbol{y}))^T \mathrm{d}\boldsymbol{w} \right) \\

\end{align}
$$

可得 $\frac{\partial l}{\partial \boldsymbol{w}} = 2 X^T (X \boldsymbol{w} - \boldsymbol{y})$。

接下来求 $\frac{\partial l}{\partial \boldsymbol{w}}$ 的零点，即

$$
\begin{align}
\frac{\partial l}{\partial \boldsymbol{w}} &= \boldsymbol{0} \\
2 X^T (X \boldsymbol{w} - \boldsymbol{y}) &= \boldsymbol{0} \\
X^T (X \boldsymbol{w} - \boldsymbol{y}) &= \boldsymbol{0} \\
X^TX \boldsymbol{w} &= X^T \boldsymbol{y} \\
\boldsymbol{w} &= (X^TX)^{-1} X^T \boldsymbol{y}

\end{align}
$$



**例5 [logistic回归]:**     $l = -\boldsymbol{y}^T \log \mathrm{softmax} (W \boldsymbol{x})$,   $l \in \mathbb{R}$,  $\boldsymbol{y} \in \mathbb{R}^{m \times 1}$,  $W \in \mathbb{R}^{m \times n}$,  $\boldsymbol{x} \in \mathbb{R}^{n \times 1}$,  $\mathrm{softmax}(x) = \frac{\exp (x)}{\boldsymbol{1}^T \exp (x)}$,  求 $\frac{\partial l}{\partial W}$ 。

**解：**


$$
\begin{align}
\log(\boldsymbol{u} / c) &= \log(\boldsymbol{u}) - \boldsymbol{1} \log(c) \\
y^T \boldsymbol{1} &= 1 
\end{align}
$$

$$
\begin{align}
l &= -y^T \log \mathrm{softmax} (Wx) \\
&= -y^T (\log ( \exp(Wx) ) - \boldsymbol{1} \log (\boldsymbol{1}^T \exp(Wx))) \\
&= -y^T Wx +  \log (\boldsymbol{1}^T \exp(Wx)) 

\\
\\

\mathrm{d} l &= -y^T \mathrm{d}W x + \frac{ \mathrm{d} (\boldsymbol{1}^T \exp(Wx)) }{\boldsymbol{1}^T \exp(Wx)} \\
&= -y^T \mathrm{d}W x + \frac{ \boldsymbol{1}^T \mathrm{d} ( \exp(Wx)) }{\boldsymbol{1}^T \exp(Wx)} \\
&= -y^T \mathrm{d}W x + \frac{ \boldsymbol{1}^T ( \exp(Wx) \odot \mathrm{d} (Wx) )}{\boldsymbol{1}^T \exp(Wx)} \\
&= -y^T \mathrm{d}W x + \frac{ \boldsymbol{1}^T ( \exp(Wx) \odot (\mathrm{d}Wx) )}{\boldsymbol{1}^T \exp(Wx)} \\
&= -y^T \mathrm{d}W x + \frac{ (\boldsymbol{1} \odot \exp(Wx))^T \mathrm{d}Wx }{\boldsymbol{1}^T \exp(Wx)} \\
&= -y^T \mathrm{d}W x + \frac{ (\exp(Wx))^T \mathrm{d}Wx }{\boldsymbol{1}^T \exp(Wx)} \\
&= -y^T \mathrm{d}W x + \frac{ (\exp(Wx))^T  }{\boldsymbol{1}^T \exp(Wx)} \mathrm{d}Wx \\
&= -y^T \mathrm{d}W x + \mathrm{softmax}(Wx)^T \mathrm{d}Wx \\
&= ( \mathrm{softmax}(Wx)^T -y^T ) \mathrm{d}Wx \\
&= ( \mathrm{softmax}(Wx) -y )^T \mathrm{d}Wx 

\\
\\

\mathrm{d} l &= \mathrm{tr} \left( \mathrm{d} l \right) \\
&= \mathrm{tr} \left( ( \mathrm{softmax}(Wx) -y )^T \mathrm{d}Wx  \right) \\
&= \mathrm{tr} \left( x ( \mathrm{softmax}(Wx) -y )^T \mathrm{d}W  \right) \\
&= \mathrm{tr} \left( (( \mathrm{softmax}(Wx) -y )x^T)^T \mathrm{d}W  \right) \\

\end{align}
$$



由此可得， $\frac{\partial l}{\partial W} = ( \mathrm{softmax}(Wx) -y )x^T$ 。



**例6 [二层神经网络]：**    $l = -y^T \log \mathrm{softmax} (W_2 \sigma(W_1 x))$,   $l \in \mathbb{R}$,  $\boldsymbol{y} \in \mathbb{R}^{m \times 1}$,  $W_1 \in \mathbb{R}^{p \times n}$,  $W_2 \in \mathbb{R}^{m \times p}$,   $\boldsymbol{x} \in \mathbb{R}^{n \times 1}$,  $\mathrm{softmax}(x) = \frac{\exp (x)}{\boldsymbol{1}^T \exp (x)}$,   $\sigma(x) = \frac{1}{1 + \exp(-x)}$,  求 $\frac{\partial l}{\partial W_1}$,   $\frac{\partial l}{\partial W_2}$  。

**解：**



令 $a = W_2 \sigma(W_1 x)$ ,


$$
\begin{align}
l &= -y^T \log \mathrm{softmax}(a) \\
&= -y^T \left( \log \exp (a) - \boldsymbol{1} \log (\boldsymbol{1}^T \exp (a)) \right) \\
&= -y^Ta + \log (\boldsymbol{1}^T \exp (a))

\\\\

\mathrm{d} l &= \mathrm{d} \left( -y^Ta + \log (\boldsymbol{1}^T \exp (a)) \right) \\
&= -y^T \mathrm{d}a + \frac{\boldsymbol{1}^T (\exp(a) \odot \mathrm{d} a)}{\boldsymbol{1}^T \exp(a)} \\
&= -y^T \mathrm{d}a + \frac{ ( \boldsymbol{1} \odot \exp(a) )^T  \mathrm{d} a}{\boldsymbol{1}^T \exp(a)} \\
&= -y^T \mathrm{d}a + \frac{ \exp(a)^T  \mathrm{d} a}{\boldsymbol{1}^T \exp(a)} \\
&= -y^T \mathrm{d}a + \mathrm{softmax}(a)^T \mathrm{d} a \\
&= \left( \mathrm{softmax}(a) -y \right)^T \mathrm{d} a 

\end{align}
$$

$$
\begin{align}

\mathrm{d} a &= \mathrm{d}( W_2 \sigma(W_1 x) ) \\
&= \mathrm{d} W_2 \sigma(W_1 x) + W_2(\sigma'(W_1) x \odot (\mathrm{d} W_1 x) ) \\
&= \mathrm{d} W_2 \sigma(W_1 x) + (W_2 \odot \sigma'(W_1)) \mathrm{d} W_1 x 

\end{align}
$$

$$
\begin{align}
\mathrm{d} l &= \mathrm{tr} \left( \mathrm{d} l \right) \\
&= \mathrm{tr} \left( \left( \mathrm{softmax}(a) -y \right)^T \mathrm{d} a \right) \\
&= \mathrm{tr} \left( \left( \mathrm{softmax}(a) -y \right)^T \left( \mathrm{d} W_2 \sigma(W_1 x) + (W_2 \odot \sigma'(W_1)) \mathrm{d} W_1 x  \right) \right) \\
&= \mathrm{tr} \left( \left( \mathrm{softmax}(a) -y \right)^T  \mathrm{d} W_2 \sigma(W_1 x)  \right) + \mathrm{tr} \left( \left( \mathrm{softmax}(a) -y \right)^T  (W_2 \odot \sigma'(W_1)) \mathrm{d} W_1 x  \right) \\
&= \mathrm{tr} \left( \sigma(W_1 x)  \left( \mathrm{softmax}(a) -y \right)^T  \mathrm{d} W_2  \right) + \mathrm{tr} \left( x \left( \mathrm{softmax}(a) -y \right)^T  (W_2 \odot \sigma'(W_1)) \mathrm{d} W_1  \right) \\
&= \mathrm{tr} \left( \left( ( \mathrm{softmax}(a) -y ) \sigma(W_1 x)^T  \right)^T  \mathrm{d} W_2  \right) + \mathrm{tr} \left( \left( (W_2 \odot \sigma'(W_1))^T ( \mathrm{softmax}(a) -y ) x^T \right)^T   \mathrm{d} W_1  \right) 
\end{align}
$$

由此可得，

$$
\begin{align}
\frac{\partial l}{\partial W_1} &= (W_2 \odot \sigma'(W_1))^T ( \mathrm{softmax}(a) -y ) x^T \\ 
&= (W_2 \odot \sigma'(W_1))^T ( \mathrm{softmax}(W_2 \sigma(W_1 x)) -y ) x^T 
\\\\
\frac{\partial l}{\partial W_2} &= ( \mathrm{softmax}(a) -y ) \sigma(W_1 x)^T  \\
&= ( \mathrm{softmax}(W_2 \sigma(W_1 x)) -y ) \sigma(W_1 x)^T
\end{align}
$$



# 矩阵对矩阵求导

## 定义

矩阵对矩阵的导数一个定义，先从向量对向量的导数定义来看

**向量对向量的导数定义：**

$f \in \mathbb{R}^{p \times 1}$,   $x \in \mathbb{R}^{m \times 1}$,   $\frac{\partial f}{\partial x} \in \mathbb{R}^{m \times p}$ 

$$
\frac{\partial f}{\partial x} = 
\begin{bmatrix}
\frac{\partial f_1}{\partial x_1} & \frac{\partial f_2}{\partial x_1} & \cdots & \frac{\partial f_p}{\partial x_1} \\
\frac{\partial f_1}{\partial x_2} & \frac{\partial f_2}{\partial x_2} & \cdots & \frac{\partial f_p}{\partial x_2} \\
\vdots & \vdots & \ddots & \vdots \\
\frac{\partial f_1}{\partial x_m} & \frac{\partial f_2}{\partial x_m} & \cdots & \frac{\partial f_p}{\partial x_m} 
\end{bmatrix}
$$

$$
\partial f = \left( \frac{\partial f}{\partial x} \right)^T \mathrm{d} x
$$

**矩阵对矩阵的导数的定义：**

矩阵对矩阵的定义需要先将矩阵向量化:

$F \in \mathbb{R}^{p \times q}$,  $X \in \mathbb{R}^{m\times n}$,   $\frac{\partial F}{\partial X} \in \mathbb{R}^{mn \times pq}$

$$
\mathrm{vec}(X) = [X_{11}, \cdots, X_{m1}, X_{12}, \cdots, X_{m2}, \cdots, X_{1n}, \cdots, X_{mn}]^T
$$

$$
\frac{\partial F}{\partial X} = \frac{\partial \mathrm{vec} (F)}{\partial \mathrm{vec} (X)} = \frac{\partial F}{\partial \mathrm{vec} (X)} =\frac{\partial \mathrm{vec} (F)}{\partial X}
$$

$$
\mathrm{vec} (\mathrm{d} F) = \left( \frac{\partial F}{\partial X} \right)^T \mathrm{vec}(\mathrm{d} X)
$$

**注：** 为了兼容上文中对 $X \in \mathbb{R}^{m \times n}$ 矩阵的导数定义，避免混淆，用记号 $\nabla_X f$ 上文中的 $\frac{\partial f}{\partial X} \in \mathbb{R}^{m \times n}$ ，则新的定义 $\frac{\partial f}{\partial X} = \mathrm{vec} (\nabla_X f) \in \mathbb{R}^{mn \times 1}$ 。



## 向量化矩阵的运算法则

1. **线性：**   $\mathrm{vec} (A+B) = \mathrm{vec} (A) + \mathrm{vec} (B)$

2. **矩阵乘法：**  $\mathrm{vec} (AXB) = (B^T \otimes A) \mathrm{vec} (X)$

3. **转置：**    $\mathrm{vec} (A^T) = K_{mn} \mathrm{vec} (A)$，  其中 $K_{mn} \in \mathbb{R}^{mn \times mn}$  是交换矩阵，将按列有限的向量化变为按行有限的向量化。

4. **逐元素乘法：**  $\mathrm{vec} (A \odot X) = \mathrm{diag} ( \mathrm{vec} (A)) \mathrm{vec} (X)$,   $\mathrm{diag} ( \mathrm{vec} (A)) \in \mathbb{R}^{mn \times mn}$ 



## Kronecker乘积和交换矩阵相关的恒等式

1. $A \otimes ( B + C) = A \otimes B + A \otimes C$
2. $(A + B) \otimes C = A \otimes C + B \otimes C$
3. $(kA) \otimes B = A \otimes (kB) = k (A \otimes B)$
4. $(A \otimes B) \otimes C = A \otimes (B \otimes C)$
5. $(A \otimes B)^T = A^T \otimes B^T$
6. $(A \otimes B)^{-1} = A^{-1} \otimes B^{-1}$
7. $\mathrm{vec}(ab^T) = b \otimes a$
8. $(A \otimes B)(C \otimes D) = (AC) \otimes (BD)$
9. $K_{mn} = K_{nm}^T$
10. $K_{pm} (A \otimes B) K_{nq} = B \otimes A$,  $A \in \mathbb{R}^{m \times n}$,  $B \in \mathbb{R}^{p \times q}$



## 求解方法

**若矩阵函数 $F$ 是矩阵 $X$ 经加减乘法、逆、行列式、逐元素函数等运算构成，则使用相应的运算法则对 $F$ 求微分，再做向量化并使用技巧将其他项交换至 $\mathrm{vec}(\mathrm{d} X)$ 左侧，对照导数与微分的联系 $\mathrm{vec} (\mathrm{d} F) = \left( \frac{\partial F}{\partial X} \right)^T \mathrm{vec}(\mathrm{d} X)$ ,  即可得到导数。**

特别地，若矩阵退化为向量，对照导数与微分的联系 $\mathrm{d}f = \left( \frac{\partial f}{\partial x} \right)^T \mathrm{d} x $,  即可得到导数。



**复合：**   假设已求得 $\frac{\partial F}{\partial Y}$， 而 $Y$ 是 $X$ 的函数， 求 $\frac{\partial F}{\partial X}$ 。

**解：**

根据链式法则，

$$
\begin{align}
\mathrm{vec} (\mathrm{d} F) &= \left( \frac{\partial F}{\partial Y} \right)^T \mathrm{vec}(\mathrm{d} Y) \\
&= \left( \frac{\partial F}{\partial Y} \right)^T \left( \frac{\partial Y}{\partial X} \right)^T \mathrm{vec}(\mathrm{d} X) \\
&= \left( \frac{\partial Y}{\partial X} \frac{\partial F}{\partial Y}  \right)^T \mathrm{vec}(\mathrm{d} X) \\
\end{align}
$$


由此可得 $\frac{\partial F}{\partial X} = \frac{\partial Y}{\partial X} \frac{\partial F}{\partial Y}  $ 。



## 案例

**例1：**   $F = AX$,   $X \in \mathbb{R}^{m \times n}$,   求 $\frac{\partial F}{\partial X}$ 。

**解：**

$$
\mathrm{d} F = A \mathrm{d} X
$$

$$
\begin{align}
\mathrm{vec} (\mathrm{d} F) &= \mathrm{vec} (A \mathrm{d} X) \\
&= (I_n \otimes A) \mathrm{vec} (\mathrm{d} X) \\
&= (I_n \otimes A^T)^T \mathrm{vec} (\mathrm{d} X)
\end{align}
$$

由此可得 $\frac{\partial F}{\partial X} = I_n \otimes A^T$ 。



**例2：**  $F = A \exp(XB)$,   $A \in \mathbb{R}^{l \times m}$,   $X \in \mathbb{R}^{m \times n}$,   $B \in \mathbb{R}^{n \times p}$,   求 $\frac{\partial F}{\partial X}$ 。 

**解：**

$$
\begin{align}
\mathrm{d} F &= A \mathrm{d} ( \exp(XB) ) \\
&= A (\exp(XB) \odot (\mathrm{d} X B))

\\\\

\mathrm{vec}(\mathrm{d} F) &= \mathrm{vec} \left( A (\exp(XB) \odot (\mathrm{d} X B)) \right) \\
&= (I_p \otimes A) \mathrm{vec} \left( \exp(XB) \odot (\mathrm{d} X B) \right) \\
&= (I_p \otimes A) \mathrm{diag} (\mathrm{vec} (\exp(XB)) ) \mathrm{vec} \left( \mathrm{d} X B \right) \\
&= (I_p \otimes A) \mathrm{diag} (\mathrm{vec} (\exp(XB)) ) (B^T \otimes I_m) \mathrm{vec} \left( \mathrm{d} X \right) \\
&= \left( (B^T \otimes I_m)^T \mathrm{diag} (\mathrm{vec} (\exp(XB)) ) (I_p \otimes A)^T \right)^T  \mathrm{vec} \left( \mathrm{d} X \right) \\
&= \left( (B \otimes I_m) \mathrm{diag} (\mathrm{vec} (\exp(XB)) ) (I_p \otimes A^T) \right)^T  \mathrm{vec} \left( \mathrm{d} X \right)
\end{align}
$$

由此可得 $\frac{\partial F}{\partial X} = (B \otimes I_m) \mathrm{diag} (\mathrm{vec} (\exp(XB)) ) (I_p \otimes A^T)$ 。



**例3：[logistic 回归]**    $l = -\boldsymbol{y}^T \log \mathrm{softmax} (W \boldsymbol{x})$,   $l \in \mathbb{R}$,  $\boldsymbol{y} \in \mathbb{R}^{m \times 1}$,  $W \in \mathbb{R}^{m \times n}$,  $\boldsymbol{x} \in \mathbb{R}^{n \times 1}$,  $\mathrm{softmax}(x) = \frac{\exp (x)}{\boldsymbol{1}^T \exp (x)}$,  求 $\nabla_W l$  和 $\nabla_W^2 l$ 。

**解：**

上文标量对矩阵求导的案例中已求出 :

$$
\nabla_W l = ( \mathrm{softmax}(Wx) -y )x^T
$$

接下来求 $\nabla_W^2 l$ : 

定义 $a = Wx$  ,

$$
\begin{align}
\mathrm{d} \nabla_Wl  &= \mathrm{d}(\mathrm{softmax}(a))  x^T \\
&= \mathrm{d} \left( \frac{\exp (a)}{\boldsymbol{1}^T \exp (a)} \right) x^T \\
&= \left( \frac{\exp (a) \odot \mathrm{d} a}{\boldsymbol{1}^T \exp (a)} - \frac{\exp (a) (\boldsymbol{1}^T (\exp (a) \odot \mathrm{d} a))}{(\boldsymbol{1}^T \exp (a))^2} \right) x^T \\
&= \left( \frac{ \mathrm{diag}( \exp (a) ) \mathrm{d} a}{\boldsymbol{1}^T \exp (a)} - \frac{\exp (a) (\exp (a)^T \mathrm{d} a)}{(\boldsymbol{1}^T \exp (a))^2} \right) x^T \\
&= \left( \frac{ \mathrm{diag}( \exp (a) ) }{\boldsymbol{1}^T \exp (a)} - \frac{\exp (a) \exp (a)^T }{(\boldsymbol{1}^T \exp (a))^2} \right) (\mathrm{d} a) x^T \\
&= \left( \mathrm{diag}\left( \frac{ \exp (a) }{\boldsymbol{1}^T \exp (a)} \right)  - \left( \frac{ \exp (a) }{\boldsymbol{1}^T \exp (a)} \right) \left( \frac{ \exp (a) }{\boldsymbol{1}^T \exp (a)} \right)^T \right) (\mathrm{d} a) x^T \\
&= \left( \mathrm{diag}\left( \mathrm{softmax}(a) \right)  - (\mathrm{softmax}(a)) ( \mathrm{softmax}(a))^T \right) (\mathrm{d} a) x^T \\
\end{align}
$$

令 $D(a) = \mathrm{diag}\left( \mathrm{softmax}(a) \right)  - (\mathrm{softmax}(a)) ( \mathrm{softmax}(a))^T$ ,

$$
\begin{align}
\mathrm{d} \nabla_W l  &= D(a)(\mathrm{d} a) x^T \\
&= D(Wx) \mathrm{d} (Wx)x^T \\
&= D(Wx) \mathrm{d} W x x^T 

\\\\

\mathrm{vec}(\mathrm{d} \nabla_W l) &= \mathrm{vec}\left( D(Wx) \mathrm{d} W x x^T  \right) \\
&= \left( (x x^T)^T \otimes D(Wx) \right) \mathrm{vec}(\mathrm{d} W) \\
&= \left( (x x^T) \otimes (D(Wx))^T \right)^T \mathrm{vec}(\mathrm{d} W) \\
&= \left( (x x^T) \otimes D(Wx) \right)^T \mathrm{vec}(\mathrm{d} W) 

\\\\

\nabla_W^2 l &= (x x^T) \otimes D(Wx) \\
&= (x x^T) \otimes ( \mathrm{diag}\left( \mathrm{softmax}(Wx) \right)  - (\mathrm{softmax}(Wx)) ( \mathrm{softmax}(Wx))^T )
\end{align}
$$


# Reference

+ [矩阵求导术 (上) | 长躯鬼侠](https://zhuanlan.zhihu.com/p/24709748)
+ [矩阵求导术 (下) | 长躯鬼侠](https://zhuanlan.zhihu.com/p/24863977)