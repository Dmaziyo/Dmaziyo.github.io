---
title : '浅入Github Action'
date : 2023-10-21T10:04:48+08:00
draft : true
---
# 浅入Github Action
背景：因为第一次部署博客在GitHub上，想要实现自动化更新部署，但是由于不熟悉GitHub actions，所以写下本篇浅入一下。
## 一、什么是Github actions?
简单来说，就是由GitHub作为平台提供给开发者自动化工作流的方案，其中比较常见的工作流就是CI/CD
## 二、常见的工作流有哪些？
1. 用户提交issue -> 检测是否重要/次要 -> 是否可重现 ->分配给相应的contributer
2. 提交pull request -> review code -> 合并分支
3. 合并分支 -> 测试 -> 构建 ->部署
![常见工作流](https://github.com/Dmaziyo/blog-img/blob/main/github-aciton1.jpg?raw=true)
而action的出现就是为了自动化上述操作。
## 三、Github actions的工作原理
事件监听->执行工作流
![工作原理](https://github.com/Dmaziyo/blog-img/blob/main/github-action2.jpg?raw=true)
## 四、为什么选择Github actions?
1. 代码大部分时间是部署在GitHub上的，所以无需接入其他第三方CI/CD tool
2. 不需要去手动安装环境，可以自定义指定环境版本
3. GitHub提供了一个官方市场，可以复用其他人的actions
![actions市场](https://www.wangbase.com/blogimg/asset/201909/bg2019091105.jpg)
## 五、workflow文件语法
```yml
name: Greeting from Mona #定义workflow名称，可选参数

# 定义trigger event
on: 
    push:
        branches:[ master ]
    pull_request:
        branches:[ master ]

jobs:
  my-job:
    name: My Job
    runs-on: ubuntu-latest #指定运行的虚拟机环境
    steps:
    - name: Print a greeting
    # 指定环境变量
      env:
        MY_VAR: Hi there! My name is
        FIRST_NAME: Mona
        MIDDLE_NAME: The
        LAST_NAME: Octocat
      run: |
        echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
```
本次部署文档时使用的workflow如下
```yml
name: Deploy Hugo Blog
on:
  push:
    branches:
      - main # Replace with your branch name

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2  # 官方自带的actions,fetch the repo

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.119.0' # Replace with your Hugo version

      - name: Build
        run: hugo --minify

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```
参考资料：
https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html
https://www.youtube.com/watch?v=R8_veQiYBjI