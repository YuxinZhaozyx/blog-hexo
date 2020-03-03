---
title: 发布第一个python包到PyPI
tags: ["python"]
categories: ["tools"]
reward: true
copyright: true
date: 2020-03-01 15:04:41
thumbnail: publish-first-python-package-to-pypi/logo.png
---





本文介绍如何将python包发布到PyPI上。

<!--more-->

## 关于`setup.py`

+ 确保你当前的根目录下能找到`setup.py`。
+ `version` 最好符合语义化版本号规则，如1.2.0

**模板:**

```python
from setuptools import setup, find_packages

setup(
	name = '<your package name>',
    version = '0.1.0',
    author = '<your name>',
    author_email = 'your@your_mail',
    packages = find_packages(),
    url = 'https://github.com/<your repo>',
    license = 'LICENSE',  # refer to your LICENSE file
    description = '<short description>',
    long_description = open('README.md').read(),
    python_requires = '>=3',
    install_requires = [
        "<package required> >= 1.3.0",
    ]
)
```

## 本地打包

```shell
python setup.py sdict bdist_wheel
```

该命令会在本地建立`dict`目录用于存放生成的包。

+ `sdict` 会生成一个包含源码的包，用户获取后会在本地重新编译构建。
+ `bdist_wheel` 会生成一个预编译的`.whl` 文件。

## 创建PyPI账号

分别在[官网](https://pypi.org)和[另一用于测试的网站](https://test.pypi.org)注册一个账号，记得就算成功登陆了账号也要到注册的邮箱点开那个激活的邮件。

## 创建用户验证文件

在用户目录下新建文件 `~/.pypirc`，加入以下内容:

```ini
[distutils]
index-servers=
	pypi
	testpypi
	
[pypi]
repository = https://upload.pypi.org/legacy/
username = <username in pypi.org>

[testpypi]
repository = https://test.pypi.org/legacy/
username = <username in test.pypi.org>
```

## 上传到测试网站

首先安装一个工具[twine](https://github.com/pypa/twine)。

```shell
pip install twine
```

接下来可以通过twine上传生成的包文件。

```
twine upload -r testpypi dist/*
# or
twine upload --repository-url https://test.pypi.org/legacy/ dist/*
```

下载安装一下试试：

```
pip install -i https://test.pypi.org/simple <your package>
```

确认没有问题、能够正常使用后我们就可以进行下一步了。

## 上传到PyPI

步骤同测试网站，但是目标地址换成了 `https://upload.pypi.org/legacy/`。

```
twine upload -r pypi dist/*
# or
twine upload --repository-url https://upload.pypi.org/legacy/ dist/*
```

接下来就可以愉快地从PyPI安装了。

```
pip install <your package>
```

