---
title: 使用Github Action + Hugo 实现自动化部署博客
description: Post with featured image.
toc: true
authors: 
  - feifei-author
tags:
  - 前端
categories:
  - 分享
series:
  - 博客搭建
date: '2021-04-16'
lastmod: '2021-04-16'
draft: false
---

## 前言

很早就想部署一个自己的博客，闲暇时间记载一下所学所思，了解到hugo能快速部署一个博客，便根据文档一步一步来操作，现在记录一下！

## 一、快速开始博客搭建

### step1: 下载hugo

这里介绍windows中操作： 

- [下载链接](https://github.com/gohugoio/hugo/releases)

- 下载zip文件，解压到指定目录，比如`D:\Software\hugo_0.82.0`
- 将以上解压目录添加至环境变量中
- 在命令行工具中输入`hugo version`有版本信息输入说明操作成功

### step2: 建立新网站

```
hugo new site myBlog
```

### step3: 添加主题

[主题地址](https://themes.gohugo.io/)

```js
cd my Blog
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke

//将主题添加到站点配置中
echo theme = \"ananke\" >> config.toml
```

### step4: 添加一个博客

`hugo new posts/my-first-post.md`

可以编辑新创建的内容文件

```js
---
title: "My First Post"
date: 2019-03-26T08:47:11+01:00
draft: true
---
    这下面是blog正文
```

完成blog后，将draft更新为false，更多信息[在这里](https://gohugo.io/getting-started/usage/#draft-future-and-expired-content)

### step5: 启动Hugo服务器

`hugo server -D`

### step6: 建立静态页面

`hugo -D`

会生成./publish目录，存放的是页面的静态文件

## 二、Git仓库配置

- 在git上创建新的远程仓库命名为xxx.github.io
- 创建.gitignore文件，写入publish，这个文件夹内容不必上传，git action会帮你构建，或者在上面不生成publish文件也可以
- 在项目目录下打开命令行工具
  - `git add .`
  - `git commit -m 'fitst commit'`
  - `git remote add origin master xxx.github.io`
  - `git push -u origin master`

- 去git远程仓库上查看文件是否都已上传
- 新建一个gh-pages分支，这里存放静态页面

## 三、通过Github Action自动化部署

- 在git远端仓库创建一个[Token](https://github.com/settings/tokens)，记得在页面关闭前保存下来，然后在远端这个项目仓库中在Settings/Secrets创建一个新的Secrets，我这里是命名为ACTIONS_DEPLOY_KEY

- 在本地的myBlog项目中创建.github/workflows/deploy.yml，文件内容如下

  ```yam
  name: Deploy Hugo #自动化的名称
  
  on:
    push: # push的时候触发工作流
      branches: #触发的分支 
        - master
  
  jobs: #开始工作流
    build-deploy:
      runs-on: ubuntu-latest #使用Ubuntu镜像
      steps:
        - name: Checkout #步骤名称
          uses: actions/checkout@master #使用的软件名称和对应版本
          with: #软件内指定的参数
            submodules: true
  
        - name: Setup Hugo #安装Hugo
          uses: peaceiris/actions-hugo@v2
          with:
            hugo-version: latest
            extended: true
  
        - name: Build
          run: hugo --minify
  
        - name: Deploy
          uses: peaceiris/actions-gh-pages@v3
          with:
            personal_token: ${{ secrets.ACTIONS_DEPLOY_KEY }} # 这里的 ACTIONS_DEPLOY_KEY 则是上面设置 Private Key的变量名
            PUBLISH_BRANCH: gh-pages
            PUBLISH_DIR: ./public
            commit_message: ${{ github.event.head_commit.message }}
  ```

- 重新将本地代码推送至远端，这时候Actions会自动跑上面这个脚本，等待这个任务完成，看下gh-pages分支中是否有静态文件

## 四、通过github pages访问

在项目仓库中点击Settings --> Pages--> 把Source下的分支切换成gh-pages，可以访问上面的Git Pages地址跳转到你的博客页面。