---
title: Windows上安装pycocotools报错
tags: ["error", "python"]
categories: ["windows"]
reward: true
copyright: true
date: 2020-01-06 17:23:00
thumbnail: windows-pycocotools/coco-logo.png
---





本文记录Windows上pip安装pycocotools是遇到的问题及解决办法。

<!--more-->

## 问题

常规安装操作为

```
pip install cython
pip install git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI
```

但是安装失败，出现错误信息：

```
cl: 命令行 error D8021 :无效的数值参数 "/Wno-cpp"
```

## 解决办法

1. 从[官方github仓库](https://github.com/cocodataset/cocoapi)下载源码

   ```
   git clone https://github.com/cocodataset/cocoapi.git
   ```

2. 修改安装程序源码  `cocoapi/PythonAPI/setup.py`

   删除第12行的 `-Wno-cpp` 和 `-Wno-unused-function` 两个参数。

   ```
   ext_modules = [
       Extension(
           'pycocotools._mask',
           sources=['../common/maskApi.c', 'pycocotools/_mask.pyx'],
           include_dirs = [np.get_include(), '../common'],
           extra_compile_args=['-Wno-cpp', '-Wno-unused-function', '-std=c99'],
       )
   ]
   ```

3. 保存后开始安装

   ```
   cd cocoapi/PythonAPI
   python setup.py install
   ```

   