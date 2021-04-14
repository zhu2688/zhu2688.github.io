---
title: "Hugo博客搭建与自动部署"
date: 2020-02-01T11:09:16+08:00
tags:
- hugo
- travis CI
draft: false
---

## 关于Hugo
  从去年计划开始学习go，已经拖到了一年了，准备重新开始记录一些东西，之前github博客系统用的是jekyll，顺便把它换成hugo，很早就关注hugo了，说是最快的静态网站生成器。现阶段来看唯一的不足点就是模板比较少。

- Hugo只有一个二进制文件（比如Windows里只是一个hugo.exe）
- Hugo可以将你写好的MarkDown格式的文章自动转换为静态的网页。
- Hugo内置web服务器，可以方便的用于本地调试。

## 安装hugo
Hugo虽然是go开发的，其实不需要安装Go环境。

推荐直接到(Hugo Release)[https://github.com/gohugoio/hugo/releases]页面下载对应平台的包。

也可以使用下面方式快速安装:

- MacOS
  
```
brew install go
```
- windows

```
scoop install hugo
```

## 使用hugo

### 生成站点

> hugo new site <站点名称/路径>


```
$ hugo new site ~/Sites/hugo_site
$ cd ~/Sites/hugo_site
$ ls -l
total 8
drwxr-xr-x  10 julian  staff  320 Feb 10 14:49 .
drwxr-xr-x  19 julian  staff  608 Feb 10 14:48 ..
drwxr-xr-x   3 julian  staff   96 Feb 10 14:48 archetypes
-rw-r--r--   1 julian  staff   82 Feb 10 14:48 config.toml
drwxr-xr-x   2 julian  staff   64 Feb 10 14:48 content
drwxr-xr-x   2 julian  staff   64 Feb 10 14:48 data
drwxr-xr-x   2 julian  staff   64 Feb 10 14:48 layouts
drwxr-xr-x   3 julian  staff   96 Feb 10 14:49 resources
drwxr-xr-x   2 julian  staff   64 Feb 10 14:48 static
drwxr-xr-x   2 julian  staff   64 Feb 10 14:48 themes

```
简要介绍目录结构,最主要的目录就是```content```,```themes```,目录了,
> config.toml 网站的配置文件 
> content 一般放网站内容,所有的markdown文章
> themes 一般放主题
> layouts 目录一般放网站的模板文件
> resources 放生成的静态资源

### 创建文章

```
$ hugo new post/first.md
```
执行完后，会在```content/post```目录自动生成一个MarkDown格式的```first.md```文件：

```
---
title: "First"
date: 2020-02-10T15:20:44+08:00
draft: true
---
```

下面就可以编辑```first.md```文件来写你的文章内容了

### 启动服务

文章写好之后，本地就可以启动服务预览了

```
hugo server
```

启动成功后，会输出预览地址类似```http://localhost:54918/```的地址，打开浏览器访问，会发现什么内容都没有。是的，默认什么都没有，因为没有主题。

### 使用主题
默认主题比较简单，可以去 (主题列表)[https://themes.gohugo.io] 找,也可以去github直接使用别人的主题。比如本博客就是使用别人的主题zozo,在上面修改的。

```
git clone https://github.com/imzeuk/hugo-theme-zozo themes/zozo
```

一般主题都有相对应的配置文件，目录下有，可以参考```exampleSite```目录下的```config.toml```配置

还有一种方式，直接clone别人的github仓库代码，然后修改相关配置，写文章发布就可以了，建议熟悉和了解hugo之后可以这样操作。

### 生成发布

每次新增文章之后，执行```hugo```，就会在配置文件中的```publishdir```默认值为（public）目录下生成所有静态文件
```
hugo
```

把```public```目录下发布到github 下就可以成为pages发布了，这个编译发布可以利用 [Travis CI](http://travis-ci.com) 自动编译发布

## Travis CI 部署

### CI 服务
Travis CI提供持续集成服务（Continuous Integration，简称 CI）。它可以绑定 Github 上面的项目，只要有新的代码，就会自动抓取。然后，提供一个运行环境，执行测试，完成构建，还能部署到服务器。Travis CI 的[官网](https://travis-ci.org/)用 Github 帐号注册登录。

为了给 Travis CI 接触 repo 的权限，首先去 Github -> Settings ->Developer settings -> personal access tokens 中生成一个 token。选择 public_repo，repo:status，repo_deployment 这三项权限即可。

或者 直接点击链接 [https://github.com/settings/tokens/new](https://github.com/settings/tokens/new) 新建token后，把这个token需要保存下来后使用。

### 设置

Travis 要求项目的根目录下面，必须有一个 ```.travis.yml``` 文件。这是配置文件，指定了 Travis 的行为。该文件必须保存在 Github 仓库里面，一旦代码仓库有新的 Commit，Travis 就会去找这个文件，执行里面的命令。

然后在 Travis 项目的Setting的 Environment Variables 增加一个变量，把上一步生成的token保存为一个变量，可以在```.travis.yml```文件中使用.

```
os: linux
language: go

go:
  - "1.8"  # 指定Golang 1.8

# Specify which branches to build using a safelist
# 分支白名单限制: 只有hugo分支的提交才会触发构建
branches:
  only:
    - hugo

# install:
# # 安装最新的hugo
install:
    - uname -a
    - wget https://github.com/gohugoio/hugo/releases/download/v0.55.4/hugo_0.55.4_Linux-64bit.deb
    - sudo dpkg -i hugo*.deb
    - hugo version
    - ls
    - pwd

script:
# 运行hugo命令
  - hugo

deploy:
  provider: pages # 重要，指定这是一份github pages的部署配置
  skip_cleanup: true # 重要，不能省略
  local_dir: public # 静态站点文件所在目录
  target_branch: master # 要将静态站点文件发布到哪个分支
  token: $GITHUB_TOKEN # 重要，$GITHUB_TOKEN是变量，需要在GitHub上申请、再到配置到Travis
  keep_history: true # 是否保持target-branch分支的提交记录
  on:
    branch: hugo # 博客源码的分支
```

