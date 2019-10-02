---
title: "[PyTorch学习笔记] What is PyTorch?"
tags: ["pytorch"]
categories: ["deep-learning-framework"]
reward: true
copyright: true
date: 2019-07-22 18:11:52
thumbnail: /static/image/pytorch.png
---





本文是PyTorch官方教程 [DEEP LEARNING WITH PYTORCH: A 60 MINUTE BLITZ] What is PyTorch? 的学习笔记

<!--more-->





## PyTorch 是什么?

基于Python的科学计算包，服务于以下两种场景:

- 作为NumPy的替代品，可以使用GPU的强大计算能力
- 提供最大的灵活性和高速的深度学习研究平台

## Tensor 张量

Tensors与Numpy中的 ndarrays类似，但是在PyTorch中 Tensors 可以使用GPU进行计算.

```python
from __future__ import print_function
import torch
```

**创建一个  5x5 的矩阵，但未初始化:**

```python
x = torch.empty(5, 3)
print(x)
```

```python
tensor([[9.5511e-39, 1.0102e-38, 4.6837e-39],
        [4.9592e-39, 5.0510e-39, 9.9184e-39],
        [9.0000e-39, 1.0561e-38, 1.0653e-38],
        [4.1327e-39, 8.9082e-39, 9.8265e-39],
        [9.4592e-39, 1.0561e-38, 1.0653e-38]])
```

**创建一个随机初始化的矩阵:**

```python
x = torch.rand(5, 3)
print(x)
```

```python
tensor([[0.6004, 0.9095, 0.5525],
        [0.2870, 0.2680, 0.1937],
        [0.9153, 0.0150, 0.5165],
        [0.7875, 0.7397, 0.9305],
        [0.8575, 0.1453, 0.2655]])
```

**创建一个0填充的矩阵，数据类型为`long`:**

```python
x = torch.zeros(5, 3, dtype=torch.long)
print(x)
```

```python
tensor([[0, 0, 0],
        [0, 0, 0],
        [0, 0, 0],
        [0, 0, 0],
        [0, 0, 0]])
```

**创建tensor并使用现有数据初始化:**

```python
x = torch.tensor([5.5, 3])
print(x)
```

```python
tensor([5.5000, 3.0000])
```

**根据现有的张量创建张量**。 这些方法将重用输入张量的属性，例如， dtype，除非设置新的值进行覆盖

```python
x = torch.tensor([5.5, 3])
print(x)

x = x.new_ones(5, 3, dtype=torch.double)      # new_* 方法来创建对象
print(x)

x = torch.randn_like(x, dtype=torch.float)    # 覆盖 dtype!
print(x)                                      #  对象的size 是相同的，只是值和类型发生了变化
```

```python
tensor([5.5000, 3.0000])
tensor([[1., 1., 1.],
        [1., 1., 1.],
        [1., 1., 1.],
        [1., 1., 1.],
        [1., 1., 1.]], dtype=torch.float64)
tensor([[-0.5648,  1.4639, -0.1247],
        [ 0.4187,  0.0255, -0.0938],
        [-1.2237,  0.3889,  0.9847],
        [-0.2423, -3.3706, -0.3511],
        [-1.1498, -1.1044,  0.4582]])
```

## 获取 size

使用`size`方法与Numpy的`shape`属性返回的相同，张量也支持`shape`属性

```python
x = torch.ones(5, 3)
print(x)

print("x.size(): ", x.size())
print("x.shape:  ", x.shape)
```

```python
tensor([[1., 1., 1.],
        [1., 1., 1.],
        [1., 1., 1.],
        [1., 1., 1.],
        [1., 1., 1.]])
x.size():  torch.Size([5, 3])
x.shape:   torch.Size([5, 3])
```

>  `torch.Size()`返回值是`tuple`类型，所以它支持`tuple`类型的所有操作



## 基础类型

### 基本类型

Tensor的基本数据类型有五种：

- 32位浮点型：torch.FloatTensor。 (默认)
- 64位整型：torch.LongTensor。
- 32位整型：torch.IntTensor。
- 16位整型：torch.ShortTensor。
- 64位浮点型：torch.DoubleTensor。

除以上数字类型外，还有 byte和chart型

```python
tensor = torch.tensor([3.1433223])  # tensor([3.1433])

long = tensor.long()  # tensor([3])
half = tensor.half()  # tensor([3.1426], dtype=torch.float16)
int_t = tensor.int()  # tensor([3], dtype=torch.int32)
flo = tensor.float()  # tensor([3.1433])
short = tensor.short()  # tensor([3], dtype=torch.int16)
ch = tensor.char()  # tensor([3], dtype=torch.int8)
bt = tensor.byte()  # tensor([3], dtype=torch.uint8)
```



## Operation 操作

### 加法

```python
x = torch.rand(5, 3)
y = torch.rand(5, 3)

sum = x + y                 # 加法1，操作符
sum = torch.add(x, y)       # 加法2，函数
torch.add(x, y, out=sum)    # 加法3，提供输出张量sum作为参数
```

### 替换

```python
# add x to y
y.add_(x)
```



> 任何 以``_`` 结尾的操作都会用结果替换原变量. 例如: ``x.copy_(y)``, ``x.t_()``, 都会改变 ``x``.



### 截取

```python
print(x[:, 1])
```

### view / reshape

`torch.view` 可以改变张量的维度和大小，与numpy的reshape类似

```python
x = torch.randn(4, 4)
y = x.view(16)
z = x.view(-1, 8)  #  size -1 从其他维度推断
print(x.size(), y.size(), z.size())
```

```python
torch.Size([4, 4]) torch.Size([16]) torch.Size([2, 8])
```

### 只有一个元素的张量取值

如果你有只有一个元素的张量，使用`.item()`来得到Python数据类型的数值

```python
x = torch.randn(1)
print(x)
print(x.item())
```

```python
tensor([-0.2036])
-0.203627809882164
```



> 更多操作[点击此处](https://pytorch.org/docs/stable/torch.html)



## Numpy 转换

Torch Tensor与NumPy数组**共享底层内存地址**，修改一个会导致另一个的变化。

### Torch Tensor 转换成 NumPy数组

```python
a = torch.ones(5)
print(a)

b = a.numpy()
print(b)

a.add_(1)
print(a)
print(b)
```

```python
tensor([1., 1., 1., 1., 1.])
[1. 1. 1. 1. 1.]
tensor([2., 2., 2., 2., 2.])
[2. 2. 2. 2. 2.]
```

### NumPy数组 转换成 Torch Tensor

```python
a = np.ones(5)
b = torch.from_numpy(a)
print(a)
print(b)

np.add(a, 1, out=a)
print(a)
print(b)
```

```python
[1. 1. 1. 1. 1.]
tensor([1., 1., 1., 1., 1.], dtype=torch.float64)
[2. 2. 2. 2. 2.]
tensor([2., 2., 2., 2., 2.], dtype=torch.float64)
```



> 所有的 `Tensor` 类型默认都是基于CPU， `CharTensor` 类型不支持到 NumPy 的转换.



## CUDA 张量

使用`.to` 方法 可以将Tensor移动到任何设备中

```python
x = torch.rand(1)

# is_available 函数判断是否有cuda可以使用
# torch.device 将张量移动到指定的设备中
if torch.cuda.is_available():
    device = torch.device("cuda")          # a CUDA 设备对象
    y = torch.ones_like(x, device=device)  # 直接从GPU创建张量
    x = x.to(device)                       # 或者直接使用 .to("cuda") 将张量移动到cuda中
    z = x + y
    print(z)
    print(z.to("cpu", torch.double))       # .to 也会对变量的类型做更改
```

```python
tensor([1.2840], device='cuda:0')
tensor([1.2840], dtype=torch.float64)
```

一般情况下可以使用`.cuda`方法和 `.cpu` 方法将tensor在cpu和gpu之间移动 （需要cuda设备支持）

```python
cpu_a = torch.rand(4, 3)
cpu_a.type()  # torch.FloatTensor

gpu_a = cpu_a.cuda()
gpu_a.type()  # torch.cuda.FloatTensor

cpu_b = gpu_a.cpu()
cpu_b.type()  # torch.FloatTensor
```

## 创建变量与结果变量

`.is_leaf`：记录是否是叶子节点。通过这个属性来确定这个变量的类型 在官方文档中所说的“graph leaves”,“leaf variables”，都是指像`x`,`y`这样的手动创建的、而非运算得到的变量，这些变量成为创建变量。 像`z`这样的，是通过计算后得到的结果称为结果变量。

```python
print("x.is_leaf="+str(x.is_leaf))
print("z.is_leaf="+str(z.is_leaf))
```

```python
x.is_leaf=True
z.is_leaf=False
```

