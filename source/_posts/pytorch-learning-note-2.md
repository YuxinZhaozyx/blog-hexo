---
title: "[PyTorch学习笔记] AUTOGRAD: Automatic Differentiation"
tags: ["pytorch"]
categories: ["machine-learning"]
reward: true
copyright: true
date: 2019-07-22 19:20:28
thumbnail: /static/image/pytorch.png
---



本文是PyTorch官方教程 [DEEP LEARNING WITH PYTORCH: A 60 MINUTE BLITZ] AUTOGRAD: Automatic Differentiation 的学习笔记

<!--more-->



# Autograd: 自动求导机制

PyTorch 中所有神经网络的核心是 `autograd` 包。 我们先简单介绍一下这个包，然后训练第一个简单的神经网络。

`autograd`包为张量上的所有操作提供了自动求导。 它是一个在运行时定义的框架，这意味着反向传播是根据你的代码来确定如何运行，并且每次迭代可以是不同的。

## 张量 Tensor

`torch.Tensor`是这个包的核心类。如果设置 `.requires_grad` 为 `True`，那么将会追踪所有对于该张量的操作。 当完成计算后通过调用 `.backward()`，自动计算所有的梯度， 这个张量的所有梯度将会自动积累到 `.grad` 属性。

要阻止张量跟踪历史记录，可以调用`.detach()`方法将其与计算历史记录分离，并禁止跟踪它将来的计算记录。

为了防止跟踪历史记录（和使用内存），可以将代码块包装在`with torch.no_grad()：`中。 在评估模型时特别有用，因为模型可能具有`requires_grad = True`的可训练参数，但是我们不需要梯度计算。

在自动梯度计算中还有另外一个重要的类`Function`.

`Tensor` 和 `Function`互相连接并生成一个非循环图，它表示和存储了完整的计算历史。 每个张量都有一个`.grad_fn`属性，这个属性引用了一个创建了`Tensor`的`Function`（除非这个张量是用户手动创建的，即，这个张量的 `grad_fn` 是 `None`）。

如果需要计算导数，你可以在`Tensor`上调用`.backward()`。 如果`Tensor`是一个标量（即它包含一个元素数据）则不需要为`backward()`指定任何参数， 但是如果它有更多的元素，你需要指定一个`gradient` 参数来匹配张量的形状。



> 在其他的文章中你可能会看到说将Tensor包裹到Variable中提供自动梯度计算，Variable 这个在0.41版中已经被标注为过期了，现在可以直接使用Tensor，官方文档在[这里](https://pytorch.org/docs/stable/autograd.html#variable-deprecated)



创建一个张量并设置 `requires_grad=True` 用来追踪他的计算历史

```python
x = torch.ones(2, 2, requires_grad=True)
print(x)

y = x + 2  # 对x进行操作
print(y)
print("y.grad_fn: ", y.grad_fn)  # 结果y已经被计算出来了，所以，grad_fn已经被自动生成了

z = y * y * 3  # 对y进行操作
out = z.mean()
print(z)
print(out)
```

```python
tensor([[1., 1.],
        [1., 1.]], requires_grad=True)
tensor([[3., 3.],
        [3., 3.]], grad_fn=<AddBackward0>)
y.grad_fn:  <AddBackward0 object at 0x00000202022DAE48>
tensor([[27., 27.],
        [27., 27.]], grad_fn=<MulBackward0>)
tensor(27., grad_fn=<MeanBackward0>)
```

`.requires_grad_( ... )` 可以改变现有张量的 `requires_grad`属性。 如果没有指定的话，默认输入的flag是 `False`。

```python
a = torch.ones(2, 2)
a = (a * 3) / (a - 1)
print(a.requires_grad)
a.requires_grad_(True)
print(a.requires_grad)
b = (a * a).sum()
print(b.grad_fn)
```

```python
False
True
<SumBackward0 object at 0x0000028141CBADA0>
```

## 梯度

反向传播 因为 `out`是一个纯量（scalar），`out.backward()` 等于`out.backward(torch.tensor(1))`。

```python
out.backward()
```

print gradients $\frac{d(out)}{dx}$

```python
x = torch.ones(2, 2, requires_grad=True)
y = x + 2  
z = y * y * 3  
out = z.mean()

out.backward()
print(x.grad)
```

```python
tensor([[4.5000, 4.5000],
        [4.5000, 4.5000]])
```

可以用自动求导做更多操作

```python
x = torch.randn(3, requires_grad=True)

y = x * 2
while y.data.norm() < 1000:
    y = y * 2

print(y)
gradients = torch.tensor([0.1, 1.0, 0.0001], dtype=torch.float)
y.backward(gradients)

print(x.grad)
```

```python
tensor([-790.8533,  793.1236,  307.1018], grad_fn=<MulBackward0>)
tensor([5.1200e+01, 5.1200e+02, 5.1200e-02])
```

如果`.requires_grad=True`但是你又不希望进行autograd的计算， 那么可以将变量包裹在 `with torch.no_grad()`中:

```python
print(x.requires_grad)
print((x ** 2).requires_grad)

with torch.no_grad():
    print(x.requires_grad)
    print((x ** 2).requires_grad)
```

```python
True
True
True
False
```





> **更多阅读：** autograd 和 Function 的[官方文档](https://pytorch.org/docs/autograd)



## 拓展Autograd

如果需要自定义autograd扩展新的功能，就需要扩展Function类。因为Function使用autograd来计算结果和梯度，并对操作历史进行编码。 在Function类中最主要的方法就是`forward()`和`backward()`他们分别代表了前向传播和反向传播。

一个自定义的Function需要一下三个方法：

```
__init__ (optional)：如果这个操作需要额外的参数则需要定义这个Function的构造函数，不需要的话可以忽略。

forward()：执行前向传播的计算代码

backward()：反向传播时梯度计算的代码。 参数的个数和forward返回值的个数一样，每个参数代表传回到此操作的梯度。
```

```python
# 引入Function便于扩展
from torch.autograd.function import Function

# 定义一个乘以常数的操作(输入参数是张量)
# 方法必须是静态方法，所以要加上@staticmethod 
class MulConstant(Function):
    
    @staticmethod 
    def forward(ctx, tensor, constant):
        # ctx 用来保存信息这里类似self，并且ctx的属性可以在backward中调用
        ctx.constant=constant
        return tensor *constant
    
    @staticmethod
    def backward(ctx, grad_output):
        # 返回的参数要与输入的参数一样.
        # 第一个输入为3x3的张量，第二个为一个常数
        # 常数的梯度必须是 None.
        return grad_output, None
```

定义完我们的新操作后，我们来进行测试

```python
a=torch.rand(3,3,requires_grad=True)
b=MulConstant.apply(a,5)
print("a:"+str(a))
print("b:"+str(b)) # b为a的元素乘以5
```

反向传播，返回值不是标量，所以`backward`方法需要参数

```python
b.backward(torch.ones_like(a))
```

