# Use hugo and github action to build blog


## Hugo Install
### Download binary file
Choose the file in [Hugo](https://github.com/gohugoio/hugo/releases), Example
```bash
wget https://github.com/gohugoio/hugo/releases/download/v0.71.1/hugo_0.71.1_Linux-64bit.tar.gz
```

### Decompression
```bash
tar zxvf hugo_0.71.1_Linux-64bit.tar.gz -C /usr/bin hugo
```

## Create Repository
create a repository in github which name "<username>.github.io", then download it to the local.
```bash
git clone https://github.com/<username>/<username>.github.io.git
```
create .gitignore and commit, then push it to keep master branch

checkout to hugo branch
```bash
git checkout -b hugo
```

## Build Blog
### Initial
```bash
cd blog
# because .git folder exist，need to use '--force' to build
hugo new site . --force
```
> advise commit the change here

### Config theme
Choose themes in (hugo theme)[https://themes.gohugo.io/]

Recommended themes：
- maupassant-hugo(simple and easy to use)
- LoveIt(powerful)

Example:
1. Download themes
```bash
git submodule add https://github.com/shit-hub/maupassant-hugo.git themes/maupassant-hugo
```

2. config `config.toml`
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
Use following command to create basic page:
```
hugo new about.md
hugo new archives.md
```
archives.md content:
```
---
title: "Archives"
type: archives
---
```
you can enter some personal information in about.md，then config `traft:false`

Run `hugo server` command ，then open chrome to access localhost:1313 to preview.

## Config Github Action
Github action is github's official CI/CD tools
### Build ken pair
```bash
ssh-keygen -t rsa -b 4096 -C "your_email" -f gh-pages -N ""
```

### Config Repository
1. Open the repository's settings page on the GitHub website
2. Add the newly generated SSH public key (gh-pages.pub) to `Deploy Keys` and select `Allow write access`
3. Add the private KEY (gh-pages) to the Secrets and name it "ACTIONS DEPLOY KEY"

### Config Action
create path `.github/workflow/`，then create `hugo.yml` file in the path，content as follow：
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

## Config Github Page
- In `setting->Options->Github Pages` of the project, select `master branch` in `Source`. 
- If you have a custom domain name, you can write it on the custom domain. 
> After saving, you will add a CNAME file in the master branch of the project , It will be overwritten during the next deployment, so you need to put the CNAME file in the `static` directory of Hugo, the CNAME file will be automatically generated in the next deployment

## Text 
Submit the hugo branch to github, and then check whether the action is executed. After successful generation, visit <username>.github.io/blog in chrome to view the blog content.
