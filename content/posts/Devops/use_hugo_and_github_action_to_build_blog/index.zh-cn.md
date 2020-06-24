---
title: "个人博客自动化部署:hugo+github-action"
date: 2020-06-02T08:56:54+08:00
categories: [Devops]
tags:
 - Hugo
 - Blog
draft: false
---

## hugo安装
### 下载二进制文件
到[Hugo官网](https://github.com/gohugoio/hugo/releases)选取对应的二进制包；以0.71.1版本为例：
```bash
wget https://github.com/gohugoio/hugo/releases/download/v0.71.1/hugo_0.71.1_Linux-64bit.tar.gz
```
### 解压到PATH路径hugo + github action
```bash
tar zxvf hugo_0.71.1_Linux-64bit.tar.gz -C /usr/bin hugo
```

## github配置
在github上创建名为blog的仓库，然后下载到本地
```bash
git clone https://github.com/<username>/blog.git
```
切换到存储hugo源码的分支
```bash
git checkout -b hugo
```

## 生成博客
### 初始化网站
```bash
cd blog
# 由于当前目录已存在.git文件夹，需要添加--force强制生成
hugo new site . --force
```
> 建议这里提交一次修改

### 主题配置
到(hugo theme)[https://themes.gohugo.io/]选取主题，然后按照主题指示安装配置

推荐主题：
- maupassant-hugo(简洁易用)
- LoveIt(功能强大)

以maupassant-hugo为例
1. 下载主题
```bash
git submodule add https://github.com/shit-hub/maupassant-hugo.git themes/maupassant-hugo
```

2. 配置config.toml
```yaml
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
    repo = "shit-hub/blog"
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
使用以下命令创建基本页面
```
hugo new about.md
hugo new archives.md
```
archives.md内容如下
```
---
title: "Archives"
type: archives
---
```
about.md可以填入自己的相关信息，然后将`traft`改成`false`即可

使用`hugo server`命令，然后打开浏览器访问localhost:1313即可预览，建议预览满意后提交一次修改

## 配置Github Action
Github Action是Github官方提供的CI/CD工具
### 生成密钥对
```bash
ssh-keygen -t rsa -b 4096 -C "your_email" -f gh-pages -N ""
```

### 配置项目
1. 在 GitHub 网站中打开项目的设置页面
2. 将刚生成的 ssh 公钥（gh-pages.pub）添加到 Deploy Keys 并选择 Allow write access
3. 将私钥（gh-pages）添加到 Secrets，命名为 ACTIONS_DEPLOY_KEY

### 制定Action
新建路径`.github/workflow/`，在该目录下创建`hugo.yml`，内容如下：
```yml
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

## 配置Github page
在项目的`setting->Options->Github Pages`中,`Source`选择`master branch`，如果有自定义域名，可以在custom domain上写上，保存后会在项目的master branch中添加CNAME文件，下次部署时会覆盖，需要把CNAME文件放到hugo的`static`目录中,下次部署就会自动生成CNAME文件

## 测试
提交hugo branch到github上，然后查看action是否执行，成功生成后，访问<username>.github.io/blog查看博客内容。