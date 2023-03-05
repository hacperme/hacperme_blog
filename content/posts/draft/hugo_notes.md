---
title: "弃用 wordpress，拥抱 hugo"
date: 2023-03-05T22:15:53+08:00
lastmod: 2023-03-05T22:15:53+08:00
author: ["hacper"]
tags:
    - hugo
    - wordpress
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "从 wordpress 迁移到 hugo 的笔记" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: true # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
# mermaid: true
# cover:
#     image: ""
#     caption: ""
#     alt: ""
#     relative: false

---

## 迁移到 hugo 的理由

昨天在读博主黄湘云的文章 《[ggplot2 食谱](https://xiangyun.rbind.io/2023/02/ggplot2-cookbook/) 》，顺带探索了一下整个网站，看看还有没有其他的我感兴趣的内容。他的博客主题风格简洁、清爽，我也有意将自己的网站往这类风格上修改，进一步探索便发现了 hugo 这个建站工具。 hugo 在 GitHub 上居然有 65.7k start， 看完 hugo 的介绍，是心动的感觉，于是便也上手折腾折腾。

参照 hugo 官网的 [GETTING STARTED](https://gohugo.io/categories/getting-started) 操作一番，hugo 的基本使用流程搞懂了，也正是我想要的。

我觉得使用 hugo 有这些优点，让我决定弃用 wordpress：

1. 基于 markdown 构建静态网站，原始文本文件可以使用 git 做版本管理，可以减少我备份和维护网站的时间。
2. 主题丰富，有我想要的简洁风格的主题。
3. 生成的是静态网站，加载速度快，减少服务器的负担，原来 运行wordpress 需要的 php、MySQL 等相关程序可以卸载了。
4. 借助 GitHub Action，可以做到自动化构建和部署。
5. 可以支持 rss 订阅、评论，满足我的刚需。

## 迁移步骤

1. 安装 hugo

   本地是Windows系统，参照 [Install Hugo on Window](https://gohugo.io/installation/windows/) 下载 hugo 二进制文件，添加到环境变量PATH就安装好了，很简单。

2. 选择主题

   主题选择耗费了一点时间，对比考虑了下面这几款主题。

   -  [Hugo Lithium](http://jrutheiser.com/)  长时间未更新、似乎也太简洁了。
   -  [Manis Hugo Theme](https://github.com/yursan9/manis-hugo-theme) 简洁，但字体不太好看，start 少。
   -  [Paper](https://themes.gohugo.io/themes/hugo-paper/) 主题风格喜欢，功能满足刚需，start 1.4k。
   -  [Hugo PaperMod](https://github.com/adityatelange/hugo-PaperMod/) 主题风格喜欢，功能满足刚需，基于 [Paper](https://themes.gohugo.io/themes/hugo-paper/)，start 5.7k。

   最后选择了 [Hugo PaperMod](https://github.com/adityatelange/hugo-PaperMod/) 。

3. 配置站点

   - 创建站点
   - 添加主题
   - 修改配置
   - 增加搜索
   - 增加归档
   - 增加标签

4. 自动化部署设置

   - 构建
   - 部署
   - 坑

   github action 构建和部署脚本如下：

   ```yaml
   # Sample workflow for building and deploying a Hugo site to GitHub Pages
   name: Deploy Hugo site to Pages
   
   on:
     # Runs on pushes targeting the default branch
     push:
       branches: ["main"]
   
     # Allows you to run this workflow manually from the Actions tab
     workflow_dispatch:
   
   
   # Default to bash
   defaults:
     run:
       shell: bash
   
   
   ## A workflow run is made up of one or more jobs that can run sequentially or in parallel
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
         - name: Checkout
           uses: actions/checkout@v3
           with:
             submodules: recursive
         
         - name: Setup Hugo
           uses: peaceiris/actions-hugo@v2
   
         - name: Build
           run: hugo --minify
   
         - name: Setup ssh keys
           uses: webfactory/ssh-agent@v0.7.0
           with:
             ssh-private-key: |
               ${{ secrets.BLOG_DEPLOY_KEY }}
         - name: Scan public keys
           run: |
             ssh-keyscan ${{secrets.RSYNC_HOSTNAME}} >> ~/.ssh/known_hosts
         - name: Deploy to VPS
           run: |
             rsync -av --delete public/* ${{secrets.RSYNC_USERNAME}}@${{secrets.RSYNC_HOSTNAME}}:${{ secrets.BLOG_DEPLOY_PATH }} --rsync-path="sudo rsync" --owner --group --chown=www:www
   ```

   

5. 迁移wordpress 文章

   安装插件 [jekyll-exporter](https://wordpress.org/plugins/jekyll-exporter/), 可以导出所有文章、页面、以及上传的图片，导出的文章都是 markdown格式，复制到 hugo 的站点目录，稍微修改一下路径和标签就可以了，不太麻烦。

6. 配置 Utterances 评论系统

   点击 https://github.com/apps/utterances 安装 utterances，在安装的时候选择 Only select repositories，选择存储网站源文件的仓库，也可以选择新建其他仓库。

   编辑 config.yml 文件，打开评论功能

   ```yaml
   params:
     comments: true
   ```

   在网站根目录下创建layouts/partials/comments.html，添加以下内容：

   ```html
   {{- /* Comments area start */ -}}
   {{- /* to add comments read => https://gohugo.io/content-management/comments/ */ -}}
   <script src="https://utteranc.es/client.js"
           repo="hacperme/hacperme_blog"
           issue-term="pathname"
           label="comment"
           theme="github-light"
           crossorigin="anonymous"
           async>
   </script>
   {{- /* Comments area end */ -}}
   ```

   

## 参考文档

- [Utterances 给 Hugo PaperMod 主题添加评论系统](https://reid00.github.io/posts/utterances-%E7%BB%99-hugo-papermod-%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E8%AF%84%E8%AE%BA%E7%B3%BB%E7%BB%9F/)
- [Hugo Documentation](https://gohugo.io/documentation/)
- [PaperMod's wiki](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [PaperMod主题配置](https://shaohanyun.top/posts/env/blog_build2/)
- [Hugo的PaperMod主题踩坑历程](https://freepencil.com/2021/10/hugo-papermod%E8%B8%A9%E5%9D%91%E5%8E%86%E7%A8%8B/)
- [有哪些好看的Hugo主题？Hugo框架主题选那个? Github的star数量告诉您](https://www.andbible.com/post/hugo-theme-review-loveit-papermond-others/)

