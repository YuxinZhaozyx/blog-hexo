---
title: Support Vector Machine 支持向量机
tags: ["SVM", "math"]
categories: ["machine-learning"]
reward: true
copyright: true
date: 2019-10-11 17:06:07
thumbnail:
---





本文介绍用支持向量机 (Support Vector Machine, SVM) 解决二分类问题。

<!--more-->

# 问题

给定训练样本集 $ D = \{(\boldsymbol{x_1}, y_1), \dots ,(\boldsymbol{x_m}, y_m) \}$ , $\boldsymbol{x_i} \in \mathbb{R}^n$,  $y_i \in \{ -1 , +1 \}$ ， 在训练集的样本空间中找到一个超平面将不同类别的样本区分开。

# 线性支持向量机

## 建模

划分超平面可用一下线性方程表示:

$$
\boldsymbol w^T \boldsymbol x + b = 0
$$

假设超平面 $(w,b)$ 能将训练样本正确分类，即对于 $(x_i, y_i) \in D$ ，

$$
\begin{cases}
\boldsymbol w^T \boldsymbol x_i + b \ge +1 ,& y_i = +1 \\
\boldsymbol w^T \boldsymbol x_i + b \le -1 ,& y_i = -1 
\end{cases}
$$

我们可以将上式整理一下，得到约束条件

$$
y_i (\boldsymbol w^T \boldsymbol x_i + b) \ge 1
$$

距离超平面最近的样本点能够使上述不等式的等号成立，即刚好在正确分类的边缘，这样的样本点被称为**支持向量(support vector)**。

样本空间中任一点到超平面 $(w,b)$ 的距离可写成

$$
r = \frac{| \boldsymbol w^T \boldsymbol x+b |}{\| \boldsymbol w \|}
$$

超平面两边的两个支持向量 $x_1$, $x_2$ 到屏幕的距离被成为**间隔(margin)**:

$$
\begin{align}
\gamma &= r_1+r_2 \\
&= \frac{| \boldsymbol w^T \boldsymbol x_1+b |}{\| \boldsymbol w \|} + \frac{|\boldsymbol w^T \boldsymbol x_2+b |}{\| \boldsymbol w \|} \\
&= \frac{| +1 |}{\| \boldsymbol w \|} + \frac{| -1 |}{\| \boldsymbol w \|} \\
&= \frac{2}{\| \boldsymbol w \|}
\end{align}
$$

于是我们需要做的事情就是在满足约束条件 $y_i (w^T x_i + b) \ge 1$ 的同时，最大化这两个支持向量间的距离 $\gamma$， 即

$$
\max_{w,b} \frac{2}{\| \boldsymbol w \|} \\
s.t. \quad y_i( \boldsymbol w^T \boldsymbol x_i +b) \ge 1 , \quad i=1,2, \cdots, m
$$

但是这个最大化间隔 $\gamma = \frac{2}{\| w \|}$ 并不好优化，于是我们转而用一个等价的方式来代替,

$$
\min_{w,b} \frac{\| \boldsymbol w \|^2}{2} \\
s.t. \quad y_i(\boldsymbol w^T \boldsymbol x_i +b) \ge 1 , \quad i=1,2, \cdots, m
$$

## 求解

构造拉格朗日函数

$$
L(\boldsymbol w,b, \boldsymbol \alpha) = \frac 1 2 ||\boldsymbol w||^2 + \sum_{i=1}^m \alpha_i (1-y_i(\boldsymbol{w^T x_i}+b))
$$

其中$\alpha_i$ 为拉格朗日乘子，且$\alpha_i  \ge 0$ 。令

$$
\theta(\boldsymbol w) = \max_{\boldsymbol \alpha:\alpha_i \ge 0} L(\boldsymbol w,b,\boldsymbol \alpha)
$$

当样本解不满足约束条件，即在可行解区域外时：

$$
y_i(\boldsymbol{w^T x_i}+b) < 1
$$

此时,将$\alpha_i$设置为无穷大，则$\theta(\boldsymbol w)$也为无穷大。

当样本解满足约束条件，即在可行解区域内时：

$$
y_i(\boldsymbol{w^T x_i}+b) \ge 1
$$

此时，$\theta(\boldsymbol w)$为原函数本身。

综合以上两种情况，

$$
\theta(\boldsymbol w) =
\begin{cases}
\frac 1 2 ||\boldsymbol w||^2 &,\boldsymbol x \in可行解区域 \\
+ \infty &,\boldsymbol x \notin 可行解区域
\end{cases}
$$

原约束问题等价于

$$
\min_{\boldsymbol w,b} \theta(\boldsymbol w) = \min_{\boldsymbol w,b} \max_{\alpha_i \ge 0} L(\boldsymbol w, b, \boldsymbol \alpha) = p^*
$$

由拉格朗日函数的对偶性，将最小和最大的位置交换得到

$$
\max_{\alpha_i \ge 0}\min_{\boldsymbol w,b} L(\boldsymbol w, b, \boldsymbol \alpha) = d^*
$$

为使$p^* = d^*$，需满足两个条件：

- 优化问题是凸优化问题
- 满足KKT条件

首先，本优化问题显然是一个凸优化问题，所以条件一满足。而要满足条件二，即要求

$$
\begin{cases}
\alpha_i  \ge 0  \\
1-y_i(\boldsymbol{w^T x_i}+b) \le 0 \\
\alpha_i (1-y_i(\boldsymbol{w^T x_i}+b)) = 0
\end{cases}
$$

对 $L(\boldsymbol w, b, \boldsymbol \alpha)$ 求偏导，得



$$
\begin{align}
\frac{\partial L}{\partial \boldsymbol w} = 0 \ & \  \Rightarrow \ \ \boldsymbol w = \sum_{i=1}^m \alpha_i y_i \boldsymbol x_i \\
\frac{\partial L}{\partial b} = 0 \ & \  \Rightarrow  \ \ 0= \sum_{i=1}^m \alpha_i y_i
\end{align}
$$

将以上二式代入$L(\boldsymbol w, b, \boldsymbol \alpha)$消去$\boldsymbol w$和$b$，得

$$
L(\boldsymbol w, b, \boldsymbol \alpha) = \sum_{i=1}^m \alpha_i - \frac 1 2 \sum_{i=1}^m \sum_{j=1}^m \alpha_i \alpha_j y_i y_j \boldsymbol{x_i^Tx_j}
$$

即

$$
\min_{\boldsymbol w,b} L(\boldsymbol w, b, \boldsymbol \alpha) = \sum_{i=1}^m \alpha_i - \frac 1 2 \sum_{i=1}^m \sum_{j=1}^m \alpha_i \alpha_j y_i y_j \boldsymbol{x_i^Tx_j}
$$

**最终目标**：

$$
\max_{\boldsymbol \alpha} \min_{\boldsymbol w,b} L(\boldsymbol w, b, \boldsymbol \alpha) = \max_{\boldsymbol \alpha} ( \sum_{i=1}^m \alpha_i - \frac 1 2 \sum_{i=1}^m \sum_{j=1}^m \alpha_i \alpha_j y_i y_j \boldsymbol{x_i^Tx_j} )  \\
s.t. \ \sum_{i=1}^m \alpha_i y_i = 0 \\
\alpha_i \ge 0, \ i=1,2, \dots,m
$$

或

$$
\min_{\boldsymbol \alpha} (  \frac 1 2 \sum_{i=1}^m \sum_{j=1}^m \alpha_i \alpha_j y_i y_j \boldsymbol{x_i^Tx_j} -  \sum_{i=1}^m \alpha_i) = \min_{\boldsymbol \alpha} (  \frac12 ||w||^2 -  \sum_{i=1}^m \alpha_i)  \\
s.t. \ \sum_{i=1}^m \alpha_i y_i = 0 \\
\alpha_i \ge 0, \ i=1,2, \dots,m
$$

## 软间隔

到目前为止我们讲的都是基于训练集数据线性可分的假设下进行的，但是实际情况下几乎不存在完全线性可分的数据，为了解决这个问题，引入了“**软间隔(soft margin)**”的概念，即允许某些点不满足约束条件 $y_i(\boldsymbol{w^T x_i}+b) \ge 1$。

采用hinge损失，将原优化问题改写为

$$
\min_{\boldsymbol w, b, \boldsymbol \xi} (\frac12||\boldsymbol w||^2+C \sum_{i=1}^m \xi_i) \\
s.t.\  y_i(\boldsymbol{w^Tx_i}+b) \ge 1-\xi_i \\
\xi_i \ge 0,\ i=1,2,\dots,m \\
C>0
$$

其中$\xi_i$为**松弛变量(slack variables)**，$\xi_i=\max(0,1-y_i(\boldsymbol{w^T x_i}+b))$ ,即**hinge损失函数**。

每个样本都有一个对应的松弛变量，表征该样本不满足约束的程度。$C$称为惩罚参数，$C$值越大，对分类的惩罚越大。

## 总结

**输入**：训练集 $ D = \{(\boldsymbol{x_1}, y_1), \dots ,(\boldsymbol{x_m}, y_m) \}$ ，其中，$\boldsymbol{x_i} \in \mathbb{R}^n , y_i \in \{ -1, +1\}, i = 1,2,\dots,m$

**输出**：分离超平面和分类决策函数

(1) 选择惩罚参数$C$，构造并求解凸二次规划问题

$$
\min_{\boldsymbol \alpha} (  \frac 1 2 \sum_{i=1}^m \sum_{j=1}^m \alpha_i \alpha_j y_i y_j \boldsymbol{x_i^Tx_j} -  \sum_{i=1}^m \alpha_i) \\
s.t. \ \sum_{i=1}^m \alpha_i y_i = 0 \\
 0 \le \alpha_i \le C, i = 1,2,\dots,m
$$

取得最优解 $ \boldsymbol{\alpha^*} = (\alpha_1^* , \alpha_2^*, \dots, \alpha_m^*)^T$

(2) 计算

$$
\boldsymbol{w^*} = \sum_{i=1}^m \alpha_i^* y_i \boldsymbol{x_i}
$$

选择 $\boldsymbol{\alpha^*}$的一个分量$\alpha_j^*$满足条件$0<\alpha_j^* < C$ ，计算

$$
b^* = y_j -\sum_{i=1}^m \alpha_i^* y_i \boldsymbol{x_i^Tx_j}
$$

(3) 求分离超平面

$$
\boldsymbol{(w^*)^Tx }+b = 0
$$

分类决策函数：

$$
f(x) = sign(\boldsymbol{(w^*)^Tx }+b )
$$

# 序列最小优化(Sequential Minimal Optimization)(SMO)

线性支持向量机的核函数为$K(\boldsymbol{x_i,x_j}) = \boldsymbol{x_i^T x_j}$ 

$$
\min_{\boldsymbol \alpha} (  \frac 1 2 \sum_{i=1}^m \sum_{j=1}^m \alpha_i \alpha_j y_i y_j K(\boldsymbol{x_i,x_j}) -  \sum_{i=1}^m \alpha_i) \\
s.t. \ \sum_{i=1}^m \alpha_i y_i = 0 \\
0 \le \alpha_i \le C, \ i=1,2, \dots,m
$$

基本思路是：**如果所有变量的解都满足此最优化问题的KKT条件，那么这么最优化问题的解就得到了。** 

令

$$
u_i = \sum_{j=1}^m \alpha_j y_j K(\boldsymbol{x_j,x_i}) +b
$$

根据KKT条件可以得出二次规划问题中$\alpha_i$的取值意义为

$$
\begin{align}
&\alpha_i = 0  \Rightarrow \xi_i>0  \Rightarrow  y_i u_i \ge 1 \Rightarrow 可行解区域内\\
&\alpha_i = C  \Rightarrow  y_i u_i = 1- \xi_i  \Rightarrow  y_i u_i \le 1  \Rightarrow 可行解区域外\\
&0<\alpha_i<C  \Rightarrow   
\begin{cases}
\xi_i = 0 \\
y_i u_i = 1- \xi_i
\end{cases} 
 \Rightarrow  y_i u_i = 1  \Rightarrow 支持向量
\end{align}
$$

而最优解需要满足KKT条件，即上述3个条件都得满足，也就是说，**如果存在不满足KKT条件的$\alpha_i$，则需要更新该$\alpha_i$，这是第一个约束条件**。违背KKT条件的$\alpha_i$用一下条件判断：

$$
\alpha_i < C \ and \ y_i u_i \le 1 \\
\alpha_i > 0 \ and \ y_i u_i \ge 1 \\
\alpha_i = 0 \ or \ a_i = C \ and \ y_i u_i = 1
$$

此外，更新的同时还要受到**第二个约束条件**的限制，即

$$
\sum_{i=1}^m \alpha_i y_i = 0
$$

因为这个条件，我们**同时更新两个$\alpha$值**，保证更新之后的值仍满足第二个约束条件，假设我们选择的两个乘子为$\alpha_1$和$\alpha_2$，其他变量$\alpha_i(i=3,4,\dots,m)$固定不变，则SMO的最优化问题可以写成

$$
\min_{\alpha_1 , \alpha_2} W(\alpha_1, \alpha_2) = \frac12 \alpha_1^2 K_{11} + \frac12\alpha_2^2K_{22}+ \alpha_1 \alpha_2 y_1 y_2 K_{12} +y_1\alpha_1 \sum_{i=3}^m y_i \alpha_i K_{i1} + y_2 \alpha_2 \sum_{i=3}^m y_i \alpha_i K_{i2} - (\alpha_1+\alpha_2)\\
s.t. \ \alpha_1 y_1 + \alpha_2 y_2 = -\sum_{i=3}^m \alpha_i y_i = \zeta
$$

其中， $K_{ij}=K(\boldsymbol{x_i,x_j})$，$\zeta$为常数，目标函数中省略了不含$\alpha_1,\alpha_2$的常数项（因为求导后等于0）。

假设初始可行解为$\alpha_1^{old} , \alpha_2^{old}$，最优解为$\alpha_1^{new},\alpha_2^{new}$，且$\alpha_2^{new}$需满足$L \le \alpha_2^{new} \le H$

$$
L = max(0, \alpha_2^{old}-\alpha_1^{old}) , H=min(C,  \alpha_2^{old}-\alpha_1^{old}+C) , y_1 \ne y_2 \\
L = max(0,  \alpha_2^{old}+\alpha_1^{old}-C) , H = min(C, \alpha_2^{old}+\alpha_1^{old}) , y_1 = y_2
$$

设$E_i$为误差项，$\eta$为学习率，

$$
E_i = \left( \sum_{j=1}^m \alpha_j K(\boldsymbol{x_j,x_i})+b \right) -y_i, i=1,2 \\
\eta = K_{11} + K_{22} - 2K_{12} = \| \phi(\boldsymbol x_1) - \phi(\boldsymbol x_2) \|^2
$$

可得（具体证明可见《统计学习方法》定理7.6的证明）未经剪辑的解为 

$$
\alpha_2^{new,unc} = \alpha_2^{old} + \frac{y_2(E_1-E_2)}{\eta}
$$

考虑约束后，最终得到经剪辑的解为

$$
\alpha_2^{new} = 
\begin{cases}
H,&\alpha_2^{new,unc}>H \\
\alpha_2^{new,unc}, &L \le \alpha_2^{new,unc} \le H \\
L, &\alpha_2^{new,unc}<L
\end{cases}
$$

继而可得

$$
\alpha_1^{new} = \alpha_1^{old} + y_1 y_2 (\alpha_2^{old}- \alpha_2^{new})
$$

至此可得SMO算法中的一个子问题的最优解$(\alpha_1^{new},\alpha_2^{new})$ 。

## 简化版SMO算法

检查训练样本中每个点$(x_i,y_i)$是否满足KKT条件，如果不满足，则它对应的$\alpha_i$可以被优化，然后随机选择另一个变量$\alpha_j$进行优化。

## 完整版SMO算法

先通过外循环来选择第一个$\alpha_i$，并且选择过程会再两种方式之间进行交替：一种方式是在所有数据集上进行单遍扫描，另一种方式则是在非边界$\alpha$实现单遍扫描。所谓非边界$\alpha$即那些$\alpha \ne 0 \  and\  \alpha \ne C$的$\alpha$。对整个数据集扫描很容易，而实现非边界扫描，首先要建立这些$\alpha$的列表，然后再对这个表进行遍历。同时，该步骤跳过那些已知的不会改变的$\alpha$值。

## 总结

（1）计算误差

$$
E_i = \left( \sum_{j=1}^m \alpha_j K(\boldsymbol{x_j,x_i})+b \right) -y_i, i=1,2
$$

（2）计算上下边界L，H

$$
L = max(0, \alpha_2^{old}-\alpha_1^{old}) , H=min(C,  \alpha_2^{old}-\alpha_1^{old}+C) , y_1 \ne y_2 \\
L = max(0,  \alpha_2^{old}+\alpha_1^{old}-C) , H = min(C, \alpha_2^{old}+\alpha_1^{old}) , y_1 = y_2
$$

（3）计算学习率

$$
\eta = K_{11} + K_{22} - 2K_{12} = \| \phi(\boldsymbol x_1) - \phi(\boldsymbol x_2) \|^2
$$

（4）更新$\alpha_2$

$$
\alpha_2^{new,unc} = \alpha_2^{old} + \frac{y_2(E_1-E_2)}{\eta} \\
\alpha_2^{new} = 
\begin{cases}
H,&\alpha_2^{new,unc}>H \\
\alpha_2^{new,unc}, &L \le \alpha_2^{new,unc} \le H \\
L, &\alpha_2^{new,unc}<L
\end{cases}
$$

（5）更新$\alpha_1$

$$
\alpha_1^{new} = \alpha_1^{old} + y_1 y_2 (\alpha_2^{old}- \alpha_2^{new})
$$

（6）更新$b$

$$
b_1^{new} =b^{old} -E_1 -y_1 K_{11}(\alpha_1^{new}-\alpha_1^{old}) - y_2 K_{12}(\alpha_2^{new}-\alpha_2^{old}) \\
b_2^{new} = b^{old}-E_1 -y_1 K_{12}(\alpha_1^{new}-\alpha_1^{old}) - y_2 K_{22}(\alpha_2^{new}-\alpha_2^{old}) \\
b^{new} = \frac{b_1^{new}+b_2^{new}}2
$$

## Python实现

```python
def loadDataSet(fileName):
	dataMat = []
	labelMat = []
	fr = open(fileName)
	for line in fr.readlines():
		lineArr = line.strip().split('\t')
		dataMat.append([float(lineArr[0]), float(lineArr[1])])
		labelMat.append(float(lineArr[2]))
	return dataMat, labelMat

def selectJrandom(i,m):
	j=i
	while j==i:
		j = int(random.uniform(0,m))
	return j

def boundAlpha(alphaj, H, L):
	if alphaj > H:
		alphaj = H
	if alphaj < L:
		alphaj = L
	return alphaj

def SMOSimple(dataMat, classLabels, C, toler, maxIter):
	dataMatrix = np.mat(dataMat)
	labelMat = np.mat(classLabels).transpose()
	b = 0
	m,n = np.shape(dataMatrix)
	alphas = np.mat(np.zeros((m,1)))
	iter = 0

	while iter < maxIter:
		alphaPairsChanged = 0
		for i in range(m):			
			Ei = float(np.multiply(alphas,labelMat).T*dataMatrix*dataMatrix[i,:].T)+ b - float(labelMat[i])
			if (labelMat[i]*Ei < -toler and alphas[i]<C) or (labelMat[i]*Ei > toler and alphas[i]>0):
				j = selectJrandom(i,m)
				Ej = float(np.multiply(alphas,labelMat).T*dataMatrix*dataMatrix[j,:].T)+ b - float(labelMat[j])
				alphaIold = alphas[i].copy()
				alphaJold = alphas[j].copy()

				if labelMat[i] != labelMat[j]:
					L = max(0, alphas[j] - alphas[i])
					H = min(C, alphas[j] - alphas[i] + C)
				else:
					L = max(0, alphas[j] + alphas[i] - C)
					H = min(C, alphas[j] + alphas[i])

				if L==H:
					print "L==H"
					continue

				eta =  dataMatrix[i,:]*dataMatrix[i,:].T + dataMatrix[j,:]*dataMatrix[j,:].T - 2.0 * dataMatrix[i,:] * dataMatrix[j,:].T 
				if eta <= 0:
					print "eta<=0"
					continue

				alphas[j] += labelMat[j] * (Ei - Ej)/eta
				alphas[j] = boundAlpha(alphas[j], H, L)
				if abs(alphas[j]-alphaJold) < 0.00001 :
					print "j not moving enough"
					continue
				alphas[i] += labelMat[j]*labelMat[i]*(alphaJold-alphas[j])

				b1 = b - Ei - labelMat[i]*(alphas[i]-alphaIold)*dataMatrix[i,:]*dataMatrix[i,:].T \
				            - labelMat[j]*(alphas[j]-alphaJold)*dataMatrix[i,:]*dataMatrix[j,:].T
				b2 = b - Ej - labelMat[i]*(alphas[i]-alphaIold)*dataMatrix[i,:]*dataMatrix[j,:].T \
				            - labelMat[j]*(alphas[j]-alphaJold)*dataMatrix[j,:]*dataMatrix[j,:].T

				if 0 < alphas[i] < C :
					b = b1
				elif 0 < alphas[j] < C :
					b = b2
				else:
					b = 0.5*(b1+b2)

				alphaPairsChanged += 1
				print "iter: %d i: %d, pairs changed %d" % (iter, i, alphaPairsChanged)

		if alphaPairsChanged == 0:
			iter += 1
		else:
			iter = 0	

		print "iteration number: %d" % iter
	return b, alphas

```

```python
class OptStruct:
	def __init__(self, dataMat, classLabels, C, toler):
		self.X = dataMat
		self.labelMat = classLabels
		self.C = C
		self.toler = toler
		self.m = np.shape(dataMat)[0]
		self.alphas = np.mat(np.zeros((self.m, 1)))
		self.b = 0
		self.eCache = np.mat(np.zeros((self.m, 2))) # 误差缓存

def calcEi(optStruct, i):
	Ei = float(np.multiply(optStruct.alphas, optStruct.labelMat).T*(optStruct.X*optStruct.X[i,:].T)) + optStruct.b - float(optStruct.labelMat[i])
	return Ei

def selectJ(i, optStruct, Ei):
	maxK = -1
	maxDeltaE = 0
	Ej = 0
	optStruct.eCache[i] = [1, Ei]
	validECacheList = np.nonzero(optStruct.eCache[:,0].A)[0]
	if len(validECacheList) > 1:
		for k in validECacheList:
			if k == i:
				continue
			Ek = calcEi(optStruct, k)
			deltaE = abs(Ei - Ek)
			if deltaE > maxDeltaE:
				maxK = k
				maxDeltaE = deltaE
				Ej = Ek
		return maxK, Ej
	else:
		j = selectJrandom(i, optStruct.m)
		Ej = calcEi(optStruct, j)
	return j, Ej

def UpdateEk(optStruct, k):
	Ek = calcEi(optStruct, k)
	optStruct.eCache[k] = [1,Ek]

def innerL(i, optStruct):
	Ei = calcEi(optStruct, i)
	if(optStruct.labelMat[i]*Ei < - optStruct.toler and optStruct.alphas[i] < optStruct.C) or (optStruct.labelMat[i]*Ei > optStruct.toler and optStruct.alphas[i] > 0):
		j, Ej = selectJ(i, optStruct, Ei)
		alphaIold = optStruct.alphas[i].copy()
		alphaJold = optStruct.alphas[j].copy()
		if optStruct.labelMat[i] != optStruct.labelMat[j]:
			L = max(0, alphaJold - alphaIold)
			H = min(optStruct.C, alphaJold - alphaIold + optStruct.C)
		else:
			L = max(0, alphaJold + alphaIold - optStruct.C)
			H = min(optStruct.C, alphaJold + alphaIold)

		if L==H:
			print "L==H"
			return 0

		eta = optStruct.X[i,:]*optStruct.X[i,:].T + optStruct.X[j,:]*optStruct.X[j,:].T - 2.0 * optStruct.X[i,:]* optStruct.X[j,:].T
		if eta <= 0:
			print "eta <= 0"
			return 0

		optStruct.alphas[j] += optStruct.labelMat[j]*(Ei-Ej)/eta
		optStruct.alphas[j] = boundAlpha(optStruct.alphas[j], H, L)
		UpdateEk(optStruct, j)		

		if abs(optStruct.alphas[j] - alphaJold) < 0.00001:
			print "j not moving enough"
			return 0
		optStruct.alphas[i] += optStruct.labelMat[j]*optStruct.labelMat[i]*(alphaJold - optStruct.alphas[j])
		UpdateEk(optStruct, i)
		b1 = optStruct.b - Ei - optStruct.labelMat[i]*(optStruct.alphas[i]-alphaIold)*optStruct.X[i,:]*optStruct.X[i,:].T \
		                      - optStruct.labelMat[j]*(optStruct.alphas[j]-alphaJold)*optStruct.X[i,:]*optStruct.X[j,:].T	
		b2 = optStruct.b - Ei - optStruct.labelMat[i]*(optStruct.alphas[i]-alphaIold)*optStruct.X[i,:]*optStruct.X[j,:].T \
		                      - optStruct.labelMat[j]*(optStruct.alphas[j]-alphaJold)*optStruct.X[j,:]*optStruct.X[j,:].T	
		if 0 < optStruct.alphas[i] < optStruct.C :
			optStruct.b = b1
		elif 0 < optStruct.alphas[j] < optStruct.C :
			optStruct.b = b2
		else:
			optStruct.b = 0.5*(b1+b2)
		
		return 1
	else:
		return 0

def SMOPlatt(dataMat, classLabels, C, toler, maxIter, kTup=('lin', 0)):
	optStruct = OptStruct(np.mat(dataMat),np.mat(classLabels).transpose(), C, toler)
	iter = 0
	entireSet = True
	alphaPairsChanged = 0
	while iter < maxIter and (alphaPairsChanged > 0 or entireSet):
		alphaPairsChanged = 0
		if entireSet:
			for i in range(optStruct.m):
				alphaPairsChanged += innerL(i, optStruct)
			print "fullSet, iter: %d i: %d, pairs changed %d" %(iter, i, alphaPairsChanged)
			iter += 1
		else:
			nonBoundIs = np.nonzero((optStruct.alphas.A > 0)*(optStruct.alphas.A < C))[0]
			for i in nonBoundIs:
				alphaPairsChanged += innerL(i, optStruct)
				print "non-bound, iter: %d i: %d, pairs changed %d" %(iter, i, alphaPairsChanged)
			iter += 1
		if entireSet:
			entireSet = False
		elif alphaPairsChanged == 0:
			entireSet = True
		print "iteration number: %d" % iter
	return optStruct.b,optStruct.alphas

```

# 径向基核函数

$$
K(\boldsymbol{x,y}) = exp(-\frac{\| \boldsymbol{x-y} \|^2}{2\sigma^2})
$$

## Python实现

```python
class OptStruct:
	def __init__(self, dataMat, classLabels, C, toler, kTuple):
		self.X = dataMat
		self.labelMat = classLabels
		self.C = C
		self.toler = toler
		self.m = np.shape(dataMat)[0]
		self.alphas = np.mat(np.zeros((self.m, 1)))
		self.b = 0
		self.eCache = np.mat(np.zeros((self.m, 2))) # 误差缓存
		self.K = np.mat(np.zeros((self.m, self.m)))
		for i in range(self.m):
			self.K[:, i] = kernelTransform(self.X, self.X[i,:], kTuple)

def calcEi(optStruct, i):
	Ei = float(np.multiply(optStruct.alphas, optStruct.labelMat).T*optStruct.K[:,i]) + optStruct.b - float(optStruct.labelMat[i])
	return Ei

def selectJ(i, optStruct, Ei):
	maxK = -1
	maxDeltaE = 0
	Ej = 0
	optStruct.eCache[i] = [1, Ei]
	validECacheList = np.nonzero(optStruct.eCache[:,0].A)[0]
	if len(validECacheList) > 1:
		for k in validECacheList:
			if k == i:
				continue
			Ek = calcEi(optStruct, k)
			deltaE = abs(Ei - Ek)
			if deltaE > maxDeltaE:
				maxK = k
				maxDeltaE = deltaE
				Ej = Ek
		return maxK, Ej
	else:
		j = selectJrandom(i, optStruct.m)
		Ej = calcEi(optStruct, j)
	return j, Ej

def UpdateEk(optStruct, k):
	Ek = calcEi(optStruct, k)
	optStruct.eCache[k] = [1,Ek]

def innerL(i, optStruct):
	Ei = calcEi(optStruct, i)
	if(optStruct.labelMat[i]*Ei < - optStruct.toler and optStruct.alphas[i] < optStruct.C) or (optStruct.labelMat[i]*Ei > optStruct.toler and optStruct.alphas[i] > 0):
		j, Ej = selectJ(i, optStruct, Ei)
		alphaIold = optStruct.alphas[i].copy()
		alphaJold = optStruct.alphas[j].copy()
		if optStruct.labelMat[i] != optStruct.labelMat[j]:
			L = max(0, alphaJold - alphaIold)
			H = min(optStruct.C, alphaJold - alphaIold + optStruct.C)
		else:
			L = max(0, alphaJold + alphaIold - optStruct.C)
			H = min(optStruct.C, alphaJold + alphaIold)

		if L==H:
			print "L==H"
			return 0

		eta = optStruct.K[i,i] + optStruct.K[j,j] - 2.0 * optStruct.K[i,j]
		if eta <= 0:
			print "eta <= 0"
			return 0

		optStruct.alphas[j] += optStruct.labelMat[j]*(Ei-Ej)/eta
		optStruct.alphas[j] = boundAlpha(optStruct.alphas[j], H, L)
		UpdateEk(optStruct, j)		

		if abs(optStruct.alphas[j] - alphaJold) < 0.00001:
			print "j not moving enough"
			return 0
		optStruct.alphas[i] += optStruct.labelMat[j]*optStruct.labelMat[i]*(alphaJold - optStruct.alphas[j])
		UpdateEk(optStruct, i)
		b1 = optStruct.b - Ei - optStruct.labelMat[i]*(optStruct.alphas[i]-alphaIold)*optStruct.K[i,i] \
		                      - optStruct.labelMat[j]*(optStruct.alphas[j]-alphaJold)*optStruct.K[i,j]
		b2 = optStruct.b - Ei - optStruct.labelMat[i]*(optStruct.alphas[i]-alphaIold)*optStruct.K[i,j] \
		                      - optStruct.labelMat[j]*(optStruct.alphas[j]-alphaJold)*optStruct.K[j,j]
		if 0 < optStruct.alphas[i] < optStruct.C :
			optStruct.b = b1
		elif 0 < optStruct.alphas[j] < optStruct.C :
			optStruct.b = b2
		else:
			optStruct.b = 0.5*(b1+b2)
		
		return 1
	else:
		return 0

def SMOPlatt(dataMat, classLabels, C, toler, maxIter, kTuple=('lin', 0)):
	optStruct = OptStruct(np.mat(dataMat),np.mat(classLabels).transpose(), C, toler, kTuple)
	iter = 0
	entireSet = True
	alphaPairsChanged = 0
	while iter < maxIter and (alphaPairsChanged > 0 or entireSet):
		alphaPairsChanged = 0
		if entireSet:
			for i in range(optStruct.m):
				alphaPairsChanged += innerL(i, optStruct)
			print "fullSet, iter: %d i: %d, pairs changed %d" %(iter, i, alphaPairsChanged)
			iter += 1
		else:
			nonBoundIs = np.nonzero((optStruct.alphas.A > 0)*(optStruct.alphas.A < C))[0]
			for i in nonBoundIs:
				alphaPairsChanged += innerL(i, optStruct)
				print "non-bound, iter: %d i: %d, pairs changed %d" %(iter, i, alphaPairsChanged)
			iter += 1
		if entireSet:
			entireSet = False
		elif alphaPairsChanged == 0:
			entireSet = True
		print "iteration number: %d" % iter
	return optStruct.b,optStruct.alphas

def kernelTransform(X, A, kTuple):
	m,n = np.shape(X)
	K = np.mat(np.zeros((m,1)))
	if kTuple[0] == 'lin': 
		K = X*A.T
	elif kTuple[0] == 'rbf':
		for j in range(m):
			deltaRow = X[j,:] - A
			K[j] = deltaRow * deltaRow.T
		K = np.exp(K/(-1*kTuple[1]**2))
	else:
		raise NameError('That Kernel is not recognized')
	return K
```

```python
def testRbf(k1 = 1.3):
	dataArr, labelArr = loadDataSet('testSetRBF.txt')
	b, alphas = SMOPlatt(dataArr, labelArr, 200, 0.0001, 10000, ('rbf', k1))
	dataMat = np.mat(dataArr)
	labelMat = np.mat(labelArr).transpose()
	supportVectorIndex = np.nonzero(alphas.A>0)[0]
	supportVectors = dataMat[supportVectorIndex]
	labelSupportVector = labelMat[supportVectorIndex]
	print 'there are %d Support Vectors' % np.shape(supportVectors)[0]
	m,n = np.shape(dataMat)
	errorCount = 0
	for i in range(m):
		kernelEval = kernelTransform(supportVectors, dataMat[i,:],('rbf',k1))
		predict = kernelEval.T * np.multiply(labelSupportVector, alphas[supportVectorIndex])+b
		if np.sign(predict) != np.sign(labelArr[i]): 
			errorCount += 1
	print 'the training error rate is: %f' %(float(errorCount)/m)

```

