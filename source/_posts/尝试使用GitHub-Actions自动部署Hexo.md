---
title: 尝试使用GitHub-Actions自动部署Hexo
date: 2019-08-28 10:20:19
categories: 博客
tags:
  - CI/CD
  - GitHub
  - Hexo
---

# 尝试使用GitHub Actions自动部署Hexo

[参考链接](https://gythialy.github.io/)。

- 生成ssh部署公私钥

```shell
ssh-keygen
```

- 为Page仓库添加公钥：
  - Github：在`仓库/Settings/Deploy keys`中添加，授予写权限
  - Coding：在`仓库/设置/部署公钥`中添加，授予写权限
- 为源码仓库添加私钥：
  - Github：在`仓库/Settings/Secrets`中添加，注意记录名称。这个功能可以放一些隐私内容，包括私钥、token、密码等等。
- 编写`workflow`，文件位置`.github/workflows/deploy.yml`

```yml
name: Node CI

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [10.x]

    steps:
    - uses: actions/checkout@v1
    - name: Use Node.js 10.x
      uses: actions/setup-node@v1
      with:
        node-version: '10.x'
    - name: env prepare
      env:
        ACTIONS_PRI: ${{secrets.ACTIONS_PRI}}
      run: |
        mkdir -p ~/.ssh/
        echo "$ACTIONS_PRI" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan github.com >> ~/.ssh/known_hosts
        ssh-keyscan git.coding.net >> ~/.ssh/known_hosts
        git config --global user.name 'SinLapis'
        git config --global user.email 'stradust0001@gmail.com'
        npm i -g hexo-cli
        npm install hexo-renderer-jade hexo-renderer-stylus --save
        npm i
    - name: gen
      run: |
        hexo g -d
```

多部署注意把域名加入`~/.ssh/known_hosts`，否则会部署失败。

