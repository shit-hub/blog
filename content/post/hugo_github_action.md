---
title: "Hugo + Github Action"
date: 2020-05-27T20:28:43+08:00
description: |
 The course about how to build blog by hugo and github action
author: "Sanjo Heron"
categories: [Blog]
tags:
 - Hugo
draft: false
---            
## hugo安装
### 下载二进制文件
到[Hugo官网](https://github.com/gohugoio/hugo/releases)选取对应的二进制包；以0.71.1版本为例：
```
wget https://github.com/gohugoio/hugo/releases/download/v0.71.1/hugo_0.71.1_Linux-64bit.tar.gz
```
### 解压到PATH路径hugo + github action
```
tar zxvf hugo_0.71.1_Linux-64bit.tar.gz -C /usr/bin hugo
```

## github配置
在github上创建名为blog的仓库，然后下载到本地
```
git clone https://github.com/<username>/blog.git
```
切换到存储hugo源码的分支
```
git checkout -b hugo
```

## 生成博客
### 初始化网站
```
cd blog
# 由于当前目录已存在.git文件夹，需要添加--force强制生成
hugo new site . --force
```
> 建议这里提交一次修改

### 主题配置
到(hugo theme)[https://themes.gohugo.io/]选取主题，然后按照主题指示安装配置， 以maupassant为例
1. 下载主题
```
git submodule add https://github.com/shit-hub/maupassant-hugo.git themes/maupassant-hugo
```

2. 配置config.toml
```
baseURL = "https://blog.shit-hub.com"
languageCode = "zh-CN"
title = "SH IT Blog"
hasCJKLanguage = true
theme = "maupassant-hugo"
enableRobotsTXT = true
PaginatePath = "page"

[author]
    name = "Sanjo Heron"
    homepage = "https://www.shit-hub.com"

[params]
    subtitle = ""
    description = ""
    keywords = "Blog"

    customCSS = ["style.extra.css"]
    customJS = ["app.extra.js"]

[params.utteranc]
    repo = "JokerQyou/comments"
    issueTerm = "url"
    theme = "github-light"

[[menu.main]]
    identifier = "categories"
    name = "Categories"
    url = "/categories/"
    weight = 1

[[menu.main]]
    identifier = "tags"
    name = "Tags"
    url = "/tags/"
    weight = 2

[[menu.main]]
    identifier = "archives"
    name = "Archives"
    url = "/archives/"
    weight = 3

[[menu.main]]
    identifier = "about"
    name = "About"
    url = "/about/"
    weight = 4
```
> 可调用`hugo server`命令，然后访问localhost:1313即可预览，建议预览满意后提交一次修改

## 配置Github Action
### 生成密钥对
```
ssh-keygen -t rsa -b 4096 -C "your_email" -f gh-pages -N ""
```

### 配置项目
1. 在 GitHub 网站中打开项目的设置页面
2. 将刚生成的 ssh 公钥（gh-pages.pub）添加到 Deploy Keys 并选择 Allow write access
3. 将私钥添加到 Secrets，命名为 ACTIONS_DEPLOY_KEY

### 制定Action
打开项目中的Action，添加新的workflow，添加以下代码
```
name: github pages

on:
  push:
    branches:
      - hugo

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
      with:
        submodules: true

    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v2.3.1
      with:
        hugo-version: '0.61.0'

    - name: Build
      run: hugo --minify

    - name: add nojekyll
      run: touch ./public/.nojekyll

    - name: Deploy
      uses: peaceiris/actions-gh-pages@v2.5.1
      env:
        ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        PUBLISH_BRANCH: master
        PUBLISH_DIR: ./public
```

## 测试
提交本地修改测试是否成功
  
