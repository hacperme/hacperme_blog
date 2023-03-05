---
id: 1957
title: '在 windows 平台上 使用 git 多仓库管理工具 repo'
date: '2021-12-27T08:00:12+08:00'
author: hacper
excerpt: '介绍在 Windows 平台上使用 repo 多仓库管理工具的几种方法。'
layout: post
guid: 'https://hacperme.com/?p=1957'
permalink: /2021/12/27/git_repo_on_windows/
views:
    - '1300'
categories:
    - 笔记
tags:
    - git
    - repo
---

![photo-1640012249194-ac74a4d7c03a.jpg](https://images.hive.blog/DQmSGXHJfKaAaVnF2r9UguGKRsASW67yL7RErYrTEpJqfTk/photo-1640012249194-ac74a4d7c03a.jpg)

## repo 介绍

一个大型项目通常会有多个仓库构成，比如 Android 项目，通过 manifest 清单（xml 文件）定义一个项目中各个 git 代码仓库的关联，而 repo 就是在这种项目组织方式下的一个用于多仓库协同开发和代码评审的一个客户端工具。

repo 是使用 python 开发实现的，需要安装 python 才能运行，repo 可以用于下载和提交代码，以及对多个代码仓库同时执行 git 命令，在他的官网上可以找到更多关于repo的介绍资料：<https://gerrit.googlesource.com/git-repo/+/HEAD/README.md>。

## Windows 系统使用 repo

原始的 Google repo 工具仅支持在 Linux 系统下运行，现在支持 python 2 和 python 3，但无法在 Windows 上使用，这使得一些在 Windows 系统环境下开发的项目使用 repo 不太方便。

如果需要在Windows下使用，可以使用第三方的 [esrlabs repo](https://github.com/esrlabs/git-repo)，按照官网介绍下载安装repo，并添加环境变量就可以使用了，目前仅支持 python2，esrlabs repo 的命令参数兼容原始的google repo，这算是 Windows 下使用 repo 的一个替代方案。如果使用的是 Windows 10，还可以在 Windows 10 的Linux 子系统下安装原始 repo，在 Linux 子系统进行多仓库管理，这是第二个方案。第三个方案是使用 go 语言开发 git-repo 替代 repo，下面介绍 git-repo 的安装和使用。

## git-repo 介绍和使用示例

git-repo 是一个使用 go 编写的仓库管理工具，与原始 repo 相比，支持 Windows、Mac 和 Linux 系统，除了git，没有其他软件依赖，兼容多数原始的 repo 命令，速度也比 python 编写的 repo 快，所以在 Windows 平台上 git-repo 算是一个很好的替代原始 repo 的选择，它的官网地址为：[https://git-repo.info/zh\_cn/](https://git-repo.info/zh_cn/)，可以了解更多信息。

1. 安装

  在 [https://git-repo.info/zh\_cn/download/](https://git-repo.info/zh_cn/download/) 下载对应平台的二进制执行文件，然后将其添加到环境变量。

  在 Windows上打开 git bash，能够正确执行 git repo version 命令说明安装好了。

  ```bash
  $ git repo version
  git-repo version 0.7.8
  git version 2.22.0.windows.1
  ```
2. 使用示例

  git-repo 在使用上是作为 git 的一个子命令 git repo 来使用的，在 git repo 后面输入命令参数。

  初始化项目：git repo init -u < manifest-url>

  ```bash
  $ git repo init -u git@gitee.com:qioixiy/manifests.git
  Warning: Permanently added the ECDSA host key for IP address '212.64.62.183' to the list of known hosts.
  remote: Enumerating objects: 17, done.
  remote: Total 17 (delta 0), reused 0 (delta 0), pack-reused 17
  Unpacking objects: 100% (17/17), done.
  From gitee.com:qioixiy/manifests
   * [new branch]      master     -> origin/master
  Note: checking out '20dd4a7db47f19b7e59f4e880a46ac9732cd4986'.
  
  You are in 'detached HEAD' state. You can look around, make experimental
  changes and commit them, and you can discard any commits you make in this
  state without impacting any branches by performing another checkout.
  
  If you want to create a new branch to retain commits you create, you may
  do so (now or later) by using -b with the checkout command again. Example:
  
    git checkout -b <new-branch-name>
  
  HEAD is now at 20dd4a7 Add trac into dev-env
  Switched to a new branch 'default'
  NOTE: Your identity is: hacper <git>
  NOTE: If you want to change this, please re-run 'git repo init' with --config-name
  NOTE: repo has been initialized in G:\Workspace\fsl
  </git></new-branch-name>
  ```

  初始化成功会有一个 .repo 隐藏目录。

  同步代码：git repo sync

  ```bash
  $ git repo sync
  remote: Enumerating objects: 315, done.
  remote: Enumerating objects: 6069, done.
  remote: Total 315 (delta 0), reused 0 (delta 0), pack-reused 315
  Receiving objects: 100% (315/315), 120.01 KiB | 278.00 KiB/s, done.
  Resolving deltas: 100% (126/126), done.
  From https://gitee.com/qioixiy/shell.profileiB | 262.00 KiB/s
   * [new branch]      master     -> gitee_qioixiy/master
  remote: Total 6069 (delta 0), reused 0 (delta 0), pack-reused 6069
  Receiving objects: 100% (6069/6069), 2.35 MiB | 497.00 KiB/s, done.
  Resolving deltas: 100% (4069/4069), done.
  From https://gitee.com/qioixiy/git-repo
   * [new branch]      main       -> gitee_qioixiy/main
   * [new branch]      master     -> gitee_qioixiy/master
  ERROR: 404: bad ssh_info response of 'http://gitee.com/ssh_info'
  NOTE: fail to check remote server, you may need to install gerrit hooks by hands
  ERROR: fail to load remotes: gitee_qioixiy
  Note: checking out 'd7399d9ca2cfe40dcee22f0ea6ce0cdd283e5a78'.
  
  You are in 'detached HEAD' state. You can look around, make experimental
  changes and commit them, and you can discard any commits you make in this
  state without impacting any branches by performing another checkout.
  
  If you want to create a new branch to retain commits you create, you may
  do so (now or later) by using -b with the checkout command again. Example:
  
    git checkout -b <new-branch-name>
  
  HEAD is now at d7399d9 update home_dot/files/.profile_priv.
  Note: checking out '6a2f4fb39073a4e2e6824d5f2f4a1cbf5fe4b766'.
  
  You are in 'detached HEAD' state. You can look around, make experimental
  changes and commit them, and you can discard any commits you make in this
  state without impacting any branches by performing another checkout.
  
  If you want to create a new branch to retain commits you create, you may
  do so (now or later) by using -b with the checkout command again. Example:
  
    git checkout -b <new-branch-name>
  
  HEAD is now at 6a2f4fb repo init: Added --no-partial-clone and made it persist. Bumped version to 2.14.
  </new-branch-name></new-branch-name>
  ```

  下载代码成功结果如下：

  ![](https://cdn.jsdelivr.net/gh/hacperme/picx_hosting@master/20211226/xxx.7dh4nlz4oew0.png)

  创建新分支：git repo start --all new\_branch\_name

  ```bash
  $ git repo start --all fea/test
  Switched to a new branch 'fea/test'
  Switched to a new branch 'fea/test'
  ```

  提交代码： git repo upload