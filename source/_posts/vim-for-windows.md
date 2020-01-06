---
title: Windows上vim的安装与配置
tags: ["vim"]
categories: ["windows"]
reward: true
copyright: true
date: 2020-01-06 14:29:15
thumbnail: vim-for-windows/bg.jpg
---





本文对我在windows平台上安装和配置vim的过程中遇到的问题以及解决措施进行记录。

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=1310288821&auto=1&height=66"></iframe>

# 安装vim

一开始我按照[GitHub官方仓库vim/vim](https://github.com/vim/vim)上的官方教程，从[vim官网下载页面](https://www.vim.org/download.php)下载并安装了vim，基本能够正常使用，但是在后续安装vim插件的过程中，发现官网编译的vim并不支持python，所有依赖于python的vim插件都无法正常使用。解决方法有二：

1. 使用第三方编译的版本
2. 将github repo的代码clone下来自己编译

为了节省时间，我直接选择了直接使用第三方编译的版本，下载地址在[此处](https://tuxproject.de/projects/vim/)。

下载后在本地创建文件夹 `vim/vim82`，此处的`vim82`仅表示我安装的是vim8.2版本，将下载的压缩包的内容放在`vim82`内，确保能找到`vim/vim82/vim.exe`主程序。接下来将`vim/vim82`路径添加到环境变量中即可通过命令行指令使用vim了：

```
vim test.py
```

# 配置vim

vim的配置在unix下是通过`.vimrc`文件的进行，而在windows下是通过`_vimrc`文件。

将 `vim/vim82/vimrc_example.vim` 复制到 `vim` 目录下并重命名为 `_vimrc`，即`vim/_vimrc`。

## 基本配置

在 `vim/_vimrc` 文件末尾添加以下配置：

```
set showcmd     " 在右下角显示输入的命令
syntax on       " 开启语法高亮
set number      " 显示行号
set relativenumber  " 显示相对行号
set wrap
set ruler
set incsearch   " 使得在用/搜索时将匹配的位置实时显示出来
set showmatch
set autoindent  " 自动缩进
set backspace=endofline,start,indent  "使得backspace可以删除回车、换行及缩进
set encoding=utf-8
set nobackup    " 不生成备份文件 *~
set nowritebackup
set noundofile  " 撤销后不保留备份文件 *.un
```

## 插件管理器 vim-plug

按照 [github仓库 junegunn/vim-plug](https://github.com/junegunn/vim-plug) 的教程安装vim。

1. 下载 [`plug.vim`](https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim) 文件到 `vim/vim82/autoload` 文件夹下

2. 新建 `vim/vim82/plugged` 文件夹

3. 修改 `vim/_vimrc`，添加如下配置

   ```
   call plug#begin('./vim82/plugged')
   
   " 中间放插件的安装指令，如
   " Plug 'preservim/nerdtree'
   
   call plug#end()
   
   " 外面放插件的配置指令，如
   " map <C-f> :NERDTreeToggle<CR>
   ```

**添加插件：**

1. 在 `vim/_vimrc` 中添加对应的插件地址，保存
2. 在vim中执行 `:PlugInstall` 安装插件，vim-plug 会到 github 上将对应插件的仓库克隆下来自动安装
3. 退出vim，重新进入即可

## 插件

### vim-airline

vim-airline 插件在底部增加一个状态栏，可以显示文件路径，文件编码，光标位置等信息

**安装指令：**

```
Plug 'vim-airline/vim-airline'
```

### Ultisnips

Ultisnips 插件可以通过 <kbd>Tab</kbd> 键触发预设的补全。

**安装指令：**

```
Plug 'SirVer/ultisnips'
Plug 'honza/vim-snippets'
```

**注意：**

该插件依赖于python，要求本机已安装python，且对版本有要求，在[vim的下载页面](https://tuxproject.de/projects/vim/) 找到编译vim使用的python版本，如3.8.1，在python官网下载对应版本的python安装，别忘了添加环境变量。

### NERDTree

NERDTree 插件允许呼出文件目录树，可以方便地在vim中查找目录，切换文件。

**安装指令：**

```
Plug 'preservim/nerdtree'
```

**配置指令：**

```
map <C-f> :NERDTreeToggle<CR>
```

该配置指令将 `<C-f>` 即 <kbd>Ctrl</kbd> + <kbd>f</kbd> 映射到 `:NERDTreeToggle<CR>` 指令上。

**使用：**

1. 按下 <kbd>Ctrl</kbd> + <kbd>f</kbd> 即可呼出/收回左侧栏的目录树

2. 在打开左侧栏目录树的情况下，通过 <kbd>Ctrl</kbd> + <kbd>w</kbd> + <kbd>h</kbd> 和  <kbd>Ctrl</kbd> + <kbd>w</kbd> + <kbd>l</kbd> 可以在左右栏中移动

3. 光标在左侧目录树的情况下，可以通过以下按键触发不同功能

   ```go
   ?: 快速帮助文档
   m: 修改文件（新建文件/移动文件/重命名文件/删除文件/复制文件）
   o: 打开一个目录或者打开文件，创建的是buffer，也可以用来打开书签
   go: 打开一个文件，但是光标仍然留在NERDTree，创建的是buffer
   t: 打开一个文件，创建的是Tab，对书签同样生效
   T: 打开一个文件，但是光标仍然留在NERDTree，创建的是Tab，对书签同样生效
   i: 水平分割创建文件的窗口，创建的是buffer
   gi: 水平分割创建文件的窗口，但是光标仍然留在NERDTree
   s: 垂直分割创建文件的窗口，创建的是buffer
   gs: 和gi，go类似
   x: 收起当前打开的目录
   X: 收起所有打开的目录
   e: 以文件管理的方式打开选中的目录
   D: 删除书签
   P: 大写，跳转到当前根路径
   p: 小写，跳转到光标所在的上一级路径
   K: 跳转到第一个子路径
   J: 跳转到最后一个子路径
   <C-j>和<C-k>: 在同级目录和文件间移动，忽略子目录和子文件
   C: 将根路径设置为光标所在的目录
   u: 设置上级目录为根路径
   U: 设置上级目录为跟路径，但是维持原来目录打开的状态
   r: 刷新光标所在的目录
   R: 刷新当前根路径
   I: 显示或者不显示隐藏文件
   f: 打开和关闭文件过滤器
   q: 关闭NERDTree
   A: 全屏显示NERDTree，或者关闭全屏
   ```

### NERDCommender

一个注释的辅助插件

**安装指令：**

```
Plug 'preservim/nerdcommenter'
```

**使用：**

```
<leader>cc   加注释
<leader>cu   解开注释

<leader>c<space>  加上/解开注释, 智能判断
<leader>cy   先复制, 再注解(p可以进行黏贴)
```

默认情况下 `<leader>` 为 <kbd>\\</kbd> 键。

### Auto Pairs

autopairs 插件能够在输入左括号时自动添加右括号。

**安装指令：**

```
Plug 'jiangmiao/auto-pairs'
```

### Markdown

以下插件添加对markdown的支持

**安装指令：**

```
Plug 'godlygeek/tabular'
Plug 'plasticboy/vim-markdown'
Plug 'mzlogin/vim-markdown-toc'
Plug 'iamcco/markdown-preview.nvim', {'do': 'cd app & yarn install'}
```

**配置指令：**

```
let g:vim_markdown_folding_disabled = 1
```

**使用：**

```
:MarkdownPreview      " 打开预览
:MarkdownPreviewStop  " 关闭预览
```

### LeaderF

LeaderF 提供模糊查找的功能

**安装指令：**

```
Plug 'Yggdroot/LeaderF', {'do': './install.bat'}
```

**配置指令：**

```
map <C-g> :LeaderfFunction<CR>
```

**使用：**

| 快捷键                     | 指令               | 说明                     |
| -------------------------- | ------------------ | ------------------------ |
| `<leader>f`                | `:LeaderfFile`     | 搜索当前目录下的文件     |
| `<leader>b`                | `:LeaderfBuffer`   | 搜索当前的Buffer中的文件 |
| `<leader>g` (自行设置映射) | `:LeaderfFunction` | 搜索当前文件的函数       |

**注意：**

要使用 `:LeaderfFunction`  功能需要先安装 [ctags](https://github.com/universal-ctags/ctags-win32/releases)，将压缩包下载后解压再添加到环境变量中即可。