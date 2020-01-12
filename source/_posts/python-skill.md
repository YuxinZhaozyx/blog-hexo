---
title: python的奇妙技巧
tags: ["python", "numpy", "pytorch"]
categories: ["programming-language"]
reward: true
copyright: true
date: 2020-01-12 14:47:26
thumbnail: python-skill/python.jpg
---





本文记录平时阅读大牛的python项目代码时看到的一些奇妙的技巧， 同时也记录一些常用(boring)的技巧，持续更新中...

本文涉及的技巧除了纯python的技巧，还包含以下常用python库的技巧：

+ numpy
+ pytorch
+ multiprocessing

<!--more-->



# numpy

## 反转 numpy array 某一维度

```python
A = np.random.rand(3,3,3)
A = A[:,:,::-1]  # 反转第三维度
A = A[:,::-1,:]  # 反转第二维度
```

该方法可以用于：

+ 反转图像通道RGB到BGR
+ 反转图像x，y轴

该方法同样使用于 pytorch, tensorflow 等库



## numpy array 与 python list 的相互转换

```python
python_list = [[1,2,3],[4,5,6]]         # a python list
numpy_array = np.array(python_list)     # list --> array
numpy_array = np.asarray(python_list)   # list --> array
python_list = numpy_array.tolist()      # array --> list
```



## `np.where()` 向量化 if-else 结构

```python
A = np.array([2, 3, 5, 6])
B = np.array([1, 2, 3, 4])
C = np.array([5, 6, 7, 8])
```

对于上述数组，假设我们想要实现以下逻辑：

```python
for i in range(A.shape[0]):
    if A[i] % 2 == 0:
        D[i] = B[i]
    else:
        D[i] = C[i]
```

使用 `np.where()` 可以向量化上述过程：

```python
D = np.where(A % 2 == 0, B, C)
```



## `np.select()` 向量化 if-elif-else 结构

```python
A = np.array([2, 4, 6, 7, 8, 9])
```

假设我们想实现以下逻辑：

```python
for i in range(A.shape[0]):
    if A[i] % 4 == 0:
        B[i] = 4
    elif A[i] % 3 == 0:
        B[i] = 3
    elif A[i] % 2 == 0:
        B[i] = 2
    else:
        B[i] = 1
```

使用 `np.select()` 可以向量化上述过程：

```python
conditions = [
    A % 4 == 0,
    A % 3 == 0,
    A % 2 == 0
]

choices = [
    4,
    3,
    2
]

B = np.select(conditions, choices, default = 1)
```

对于嵌套的 if-elif-else 结构，我们依然可以用 `np.select` 将其向量化，如：

```python
for i in range(A.shape[0]):
    if A[i] % 2 == 0:
        if A[i] <= 4:
            B[i] = 4
        elif A[i] > 8:
            B[i] = 8
        else:
            B[i] = 2
    elif A[i] % 3 == 0:
        B[i] = 3
    elif A[i] % 4 == 0:
        B[i] = 4
    else:
        B[i] = 1
```

可以将其向量化为以下形式：

```python
conditions = [
    ((A % 2 == 0) and (A <= 4)),
    ((A % 2 == 0) and (A > 8)),
    A % 2 == 0，
    A % 3 == 0,
    A % 4 == 0
]

choices = [
    4,
    8,
    2,
    3,
    4
]

B = np.select(conditions, choices, default = 1)
```



## `np.vectorize()` 向量化普通 python 函数

```python
A = np.array([1, 3, 5])
B = np.array([1, 2, 3])
```

```python
def python_func(a, b):
    if a > b:
        return a + b
    else:
        return b - a
```

假设我们想实现的逻辑如下：

```python
for i in range(A.shape[0]):
    C[i] = python_func(A[i], B[i])
```

该过程可以用 `np.vectorize()` 向量化：

```forpython
vec_func = np.vectorize(python_func)
C = vec_func(A, B)
```

`np.vectorize()` 将python函数向量化后可以达到 C 语言循环的速度。

# pytorch

## torch tensor 与 python list 的相互转换

```python
python_list = [[1,2,3],[4,5,6]]           # a python list
torch_tensor = torch.Tensor(python_list)  # list --> tensor
python_list = torch_tensor.tolist()       # tensor --> list
```

## torch tensor 与 numpy array 的相互转换

```python
numpy_array = np.array([[1,2,3],[4,5,6]])     # a numpy array
torch_tensor = torch.from_numpy(numpy_array)  # array --> tensor
numpy_array = torch_tensor.numpy()            # tensor --> array
```

**注意：** 以上转换共用一个内存，即改变其中一个的值，另一个也会改变。



# multiprocessing

## 多进程加速

对于某些需要处理大批量数据的函数，可以通过将数据拆分后在多个CPU上并行执行进行加速：

```python
from multiprocessing import Pool

def parallel_apply(data, func, num_cores=4):
    data_split = np.array_split(data, num_cores)
    pool = Pool(num_cores)
    data = np.concatenate(pool.map(func, data_split))
    pool.close()
    pool.join()
    return data
```

