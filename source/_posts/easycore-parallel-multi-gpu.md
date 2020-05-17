---
title: easycore多GPU并行加速
tags: ["easycore","python","pytorch","multiprocessing"]
categories: ["tools"]
reward: true
copyright: true
date: 2020-03-03 20:04:55
thumbnail: easycore-parallel-multi-gpu/python.jpg
---



本文介绍如何使用easycore进行多GPU并行加速。

<!--more-->

上期["python多进程及pytorch多GPU并行推断"](/python-multi-gpu-multiprocessing)我介绍如何使用python自带的`multiprocessing` 库进行加速的方案，该方案存在一定缺陷：

+ 在Windows下无法运行，仅支持fork模式。
+ 消费者传回数据量过大时无法成功传输。

以上问题通过将消费者从进程改为线程后都得到了解决。

我已将完整的代码封装进了我正在开发的一个工具库[easycore](https://github.com/Yuxinzhaozyx/easycore)，欢迎star。项目文档见[此处](https://easycore.readthedocs.io/en/latest/)，含API文档及教程。

以下介绍如何使用我的工具库easycore进行多GPU并行加速：

## 安装easycore

```shell
pip install easycore==0.2.0
```

**注：** 0.2.0版本才加入多进程并行加速工具。

国内可使用清华镜像加速：

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple easycore==0.2.0
```

## 多进程加速API

多进程加速主要涉及`easycore.common.parallel.UnorderedRunner` 和`easycore.common.parallel.OrderedRunner`两个类，详细的API接口请参见[API文档](https://easycore.readthedocs.io/en/latest/modules/easycore.common.parallel.html)。

通过继承`UnorderedRunner`或`OrderedRunner`两个类，并重写以下六个类静态方法，可以自定义一个并行运行器。


```python
@staticmethod
def producer_init(device, cfg):
    """ 
    function for producer initialization.
    
    Args:
        device (str): device for the this process.
        cfg (easycore.common.config.CfgNode): config of this process, you can use it to transfer data
            to `producer_work` and `producer_end` function.
    """
    pass

@staticmethod
def producer_work(device, cfg, data):
    """ 
    function specify how the producer processes the data.
    
    Args:
        device (str): device for this process.
        cfg (easycore.common.config.CfgNode): config of this process, you can use it to get data from
            `producer_init` function and transfer data to the next `producer_work` and `producer_end`
            function.
        data (Any): data get from input of `__call__` method.
    
    Returns:
        Any: processed data
    """
    return data

@staticmethod
def producer_end(device, cfg):
    """ 
    function after finishing all of its task and before close the process.
    
    Args:
        device (str): device for this process.
        cfg (easycore.common.config.CfgNode): config of this process, you can use it to get data
            from `producer_init` and `producer_work` function.
    """
    pass

@staticmethod
def consumer_init(cfg):
    """
    function for consumer initialization.
    
    Args:
        cfg (easycore.common.config.CfgNode): config of this process, you can use it to transfer data
            to `consumer_work` and `consumer_end` function.
    """
    pass

@staticmethod
def consumer_work(cfg, data):
    """
    function specify how the consumer processses the data from producers.
    
    Args:
        cfg (easycore.common.config.CfgNode): config of this process, you can use it to get data from
            `consumer_init` function and transfer data to the next `consumer_work` and `consumer_end`
            function.
    """
    pass

@staticmethod
def consumer_end(cfg):
    """
    function after receiving all data from producers.
    
    Args:
        cfg (easycore.common.config.CfgNode): config of this process, you can use it get data from
            `consumer_work` function.

    Returns:
        Any: processed data
    """
    return None
```

## Example 1: 平方和

通常我们可以通过以下方法实现求平方和的功能：

```python
data_list = list(range(100))
result = sum([data * data for data in data_list])

# or more simple
result = 0
for data in data_list:
    square = data * data
    result += square
```

上述实现中，我们先计算了每个元素的平方，再将它们加到一起。在这种情况下，我们可以将它拆成两个任务分别分配给生产者和消费者，从而实现并行：

```python
from easycore.common.config import CfgNode
from easycore.common.parallel import UnorderedRunner

class Runner(UnorderedRunner):
    @staticmethod
    def producer_work(device, cfg, data):
        return data * data  # calculate square of data

    @staticmethod
    def consumer_init(cfg):
        cfg.sum = 0  # init a sum variable with 0, you can use cfg to transfer data

    @staticmethod
    def consumer_work(cfg, data):
        cfg.sum += data  # add the square to the sum variable

    @staticmethod
    def consumer_end(cfg):
        return cfg.sum  # return the result you need

if __name__ == '__main__':
    runner = Runner(devices=3)  # if you specify `device with a integer`, it will use cpus.
    # You can specify a list of str instead, such as: 
    # runner = Runner(devices=["cpu", "cpu", "cpu"]) 
    
    data_list = list(range(100))  # prepare data, it must be iterable
    result = runner(data_list)  # call the runner
    print(result)

    runner.close()  # close the runner and shutdown all processes it opens.
```

## Example 2: 一个神经网络推断器

我们先在 `network.py` 定义一个简单的神经网络:

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.fc = nn.Linear(1, 3)
    
    def forward(self, x):
        x = self.fc(x)
        x = F.relu(x)
        return x
```

通过以下方法，可以将推断任务分配到四个GPU上:

```python
from easycore.common.config import CfgNode
from easycore.common.parallel import OrderedRunner
from network import Net
import torch

class Predictor(OrderedRunner):
    @staticmethod
    def producer_init(device, cfg):
        cfg.model = Net()     # init the producer with a model
        cfg.model.to(device)  # transfer the model to certain device

    @staticmethod
    def producer_work(device, cfg, data):
        with torch.no_grad():
            data = torch.Tensor([[data]])  # preprocess data
            data = data.to(device)  # transfer data to certain device
            output = cfg.model(data)  # predict
            output = output.cpu()  # transfer result to cpu
        return output

    @staticmethod
    def producer_end(device, cfg):
        del cfg.model  # delete the model when all data has been predicted.

    @staticmethod
    def consumer_init(cfg):
        cfg.data_list = []  # prepare a list to store all data from producers.

    @staticmethod
    def consumer_work(cfg, data):
        cfg.data_list.append(data)  # store data from producers.

    @staticmethod
    def consumer_end(cfg):
        data = torch.cat(cfg.data_list, dim=0)  # postprocess data.
        return data

if __name__ == '__main__':
    predictor = Predictor(devices=["cuda:0", "cuda:1", "cuda:2", "cuda:3"])  # init a parallel predictor

    data_list = list(range(100))  # prepare data
    result = predictor(data_list)  # predict
    print(result.shape)

    predictor.close()  # close the predictor when you no longer need it.
```

## Example 3: 批处理数据

通过使用一个简单的生成器或者pytorch的dataloader，可以很轻松地生成批数据并通过`UnorderedRunner`或`OrderedRunner`进行加速。

```python
from easycore.common.config import CfgNode
from easycore.torch.parallel import OrderedRunner
from network import Net
import torch

def batch_generator(data_list, batch_size):
    for i in range(0, len(data_list), batch_size):
        data_batch = data_list[i : i+batch_size]
        yield data_batch

class Predictor(OrderedRunner):

    @staticmethod
    def producer_init(device, cfg):
        cfg.model = Net()
        cfg.model.to(device)

    @staticmethod
    def producer_work(device, cfg, data):
        with torch.no_grad():
            data = torch.Tensor(data).view(-1,1)
            data = data.to(device)
            output = cfg.model(data)
            output = output.cpu()
        return output

    @staticmethod
    def producer_end(device, cfg):
        del cfg.model

    @staticmethod
    def consumer_init(cfg):
        cfg.data_list = []

    @staticmethod
    def consumer_work(cfg, data):
        cfg.data_list.append(data)

    @staticmethod
    def consumer_end(cfg):
        data = torch.cat(cfg.data_list, dim=0)
        return data

if __name__ == '__main__':
    predictor = Rredictor(devices=["cuda:0", "cuda:1"])
    
    data_list = list(range(100))
    result = predictor(batch_generator(data_list, batch_size=10))  
    print(result.shape)

    predictor.close()
```

这里，我们将`easycore.common.parallel`替换成了`easycore.torch.parallel`。这两个库有着完全相同的API，区别是`easycore.torch.parallel`使用了`torch.multiprocessing`库实现而不是`multiprocessing`。

## Example 4: 将Runner外的参数传入Runner中

通过`cfg`参数，你可以很轻松的将任何参数传入自定义的Runner中。`cfg`是一个`easycore.common.config.CfgNode`的对象，它的具体使用可以参见教程["Light weight config tools"](https://easycore.readthedocs.io/en/latest/tutorials/config.html)。

接下来我们使用“幂之和”作为案例：

```python
from easycore.common.config import CfgNode as CN
from easycore.common.parallel import UnorderedRunner

class Runner(UnorderedRunner):
    @staticmethod
    def producer_work(device, cfg, data):
        return data ** cfg.exponent  # calculate power of data with outside parameter "exponent".

    @staticmethod
    def consumer_init(cfg):
        cfg.sum = 0  # init a sum variable with 0, you can use cfg to transfer data

    @staticmethod
    def consumer_work(cfg, data):
        cfg.sum += data  # add the square to the sum variable

    @staticmethod
    def consumer_end(cfg):
        return cfg.sum  # return the result you need

if __name__ == '__main__':
    # set parameters outside.
    cfg = CN()
    cfg.exponent = 3

    runner = Runner(devices=3, cfg=cfg)  # transfer `cfg` into the runner 
    
    data_list = list(range(100))
    result = runner(data_list)
    print(result)

    runner.close()
```

## API 文档

+ [easycore.common.parallel](https://easycore.readthedocs.io/en/latest/modules/easycore.common.parallel.html)
+ [easycore.torch.parallel](https://easycore.readthedocs.io/en/latest/modules/easycore.torch.parallel.html)


