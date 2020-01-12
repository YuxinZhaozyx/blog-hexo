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

