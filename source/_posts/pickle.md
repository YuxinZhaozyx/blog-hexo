---
title: Pickle Python对象序列化
tags: ["python", "pickle"]
categories: ["programming-language"]
reward: true
copyright: true
date: 2019-10-20 00:50:03
thumbnail:
---



pickle 是一个用于序列化 Python 对象的库，它可以序列化任何对象并将其保存到文件中，需要时再将其从文件中读出。本文将介绍如何使用 pickle 序列化 python 对象。
<!--more-->



# 保存python对象到文件

```python
import pickle

obj = {"pickle": "test"}  # 可以是任何对象，包括自定义的类对象

with open('file.pkl', 'wb') as f:
    pickle.dump(obj, f)  # 将obj序列化后保存到文件f中，f只要是有write方法的对象都可以
```

# 从文件读取python对象

```python
import pickle

with open('file.pkl', 'rb') as f:
    obj = pickle.load(f)  # 从f中读取obj, 如果读取的是自定义的类对象，则要求读取时已经对该类进行定义
```

# 参考资料

+ [官方文档](https://docs.python.org/2/library/pickle.html)
+ [Python Wiki](https://wiki.python.org/moin/UsingPickle)

