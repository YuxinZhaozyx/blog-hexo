---
title: GitHub Action自动化部署hexo博客到GitHub Page和Coding Page
tags: []
categories: []
reward: true
copyright: true
date: 2020-03-07 05:19:05
thumbnail: github-action-auto-deploy-hexo/logo.jpg
---



由于国内访问GitHub很慢，我的GitHub Page有时需要很久才能加载出来，刚好最近捣鼓了以下GitHub Action，便尝试了用GitHub Action实现对GitHub Page和Coding Page的自动化部署，将博客同时部署到GitHub和Coding，再利用域名DNS将两个国内和国外的访问分流到Coding和GitHub上。

<!--more-->

## GitHub准备工作

打开[GitHub Settings > Developer settings > Personal access tokens](https://github.com/settings/tokens)，添加一个新的token。

记录下这个token的值，此处我们将其称为`GH_TOKEN`，稍后会用到。

## Coding准备工作

打开Coding，新建一个跟你用户名同名的项目。

打开该项目的设置 > 功能开关，开启"构建与部署"

回到项目页面，即可在左侧工具栏找到 构建与部署 > 静态网站，自行设置即可。

接下来最关键的是获取这个仓库的访问token：打开项目的设置 > 开发者选项 > 项目令牌，创建一个新的令牌，你会获得一个用户名和一个密码，我们将`<用户名>:<密码>`记录为`CD_TOKEN`。

## GitHub Action设置

打开博客的源码仓库，在Settings > Secrets中增加以下键值对：

| 键          | 值                                                           |
| ----------- | ------------------------------------------------------------ |
| `GIT_NAME`  | 你的git设置的用户名，一般为你的github用户名                  |
| `GIT_EMAIL` | 你的git设置的邮箱                                            |
| `GH_TOKEN`  | 在github获得个人访问token                                    |
| `GH_REF`    | 用于部署的github仓库的git地址，格式为`<user-name>/<repo-name>.git` |
| `CD_TOKEN`  | 在coding获得的仓库访问token，格式为`<token-user-name>:<token-password>` |
| `CD_REF`    | 用于部署的github仓库的git地址，格式为`<user-name>/<repo-name>.git` |

添加GitHub Action:

```yaml
name: 自动部署 Hexo

on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [10.x]

    steps:
      - name: 开始运行
        uses: actions/checkout@v1

      - name: 设置 Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}

      - name: 安装 Hexo CI
        run: |
          export TZ='Asia/Shanghai'
          npm install hexo-cli -g
      - name: 缓存
        uses: actions/cache@v1
        id: cache-dependencies
        with:
          path: node_modules
          key: ${{runner.OS}}-${{hashFiles('**/package-lock.json')}}

      - name: 安装插件
        if: steps.cache-dependencies.outputs.cache-hit != 'true'
        run: |
          npm install
      - name: 部署博客
        run: |
          mkdir themes
          git clone https://github.com/<user-name>/<theme-repo>.git themes/inside  #克隆hexo主题仓库到主题目录下
          hexo clean && hexo g
          cd ./public
          git init
          git config user.name "${{secrets.GIT_NAME}}"
          git config user.email "${{secrets.GIT_EMAIL}}"
          git add .
          git commit -m "Update"
          git push --force --quiet "https://${{secrets.GH_TOKEN}}@${{secrets.GH_REF}}" master:master
          git push --force --quiet "https://${{secrets.CD_TOKEN}}@${{secrets.CD_REF}}" master:master
```



## DNS设置

到自己域名的服务商修改DNS，增加两个CNAME记录，一个将你的域名映射至GitHub Page的 `<user-name>.github.io`并设置线路为境外，一个将域名映射至Coding Page的`<user-name>.coding-pages.com`并设置线路为境内。



## 参考资料

+ [Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
+ [travisci/GitHub Actions部署GitHub Pages和Coding Pages](https://zhuanlan.zhihu.com/p/103618527)