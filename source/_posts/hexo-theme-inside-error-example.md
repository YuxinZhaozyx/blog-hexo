---
title: hexo-theme-inside错误样例
tags: []
categories: ["example"]
reward: false
copyright: true
date: 2019-10-02 18:40:37
thumbnail:
---





本文仅用作 hexo-theme-inside 主题及其插件 hexo-filter-mathjax-ssr 的错误样例展示。

<!--more-->


# go语言代码块显示错误

[issue](https://github.com/ikeq/hexo-theme-inside/issues/146)

`````
## 变量

```go
var variablename type
var vname1, vname2, vname3 type 
var vname1, vname2, vname3 type = value1, value2, value3
var vname1, vname2, vname3 = value1, value2, value3
vname1, vname2, vname3 := value1, value2, value3 // 只能用在函数内部，局部变量
var(
   vname1 type
   vname2 type
) // 只能用在函数外，全局变量
```

## 常量

```go
const constantName type = value
const constantName = value
const(
	name1 = value1
    name2 = value2
)
```

`````

**结果：**

## 变量

```go
var variablename type
var vname1, vname2, vname3 type 
var vname1, vname2, vname3 type = value1, value2, value3
var vname1, vname2, vname3 = value1, value2, value3
vname1, vname2, vname3 := value1, value2, value3 // 只能用在函数内部，局部变量
var(
   vname1 type
   vname2 type
) // 只能用在函数外，全局变量
```

## 常量

```go
const constantName type = value
const constantName = value
const(
	name1 = value1
    name2 = value2
)
```
