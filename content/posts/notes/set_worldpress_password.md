---
title: "WordPress 更改文章密码保护后显示的提示内容"
date: 2015-06-13T10:43:59+08:00
lastmod: 2015-06-13T10:43:59+08:00
author: ["hacper"]
tags:
    - worldpress
categories:
    - 笔记
draft: false

description: "" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
mermaid: true
---

```php
/**
 * WordPress 更改文章密码保护后显示的提示内容
 * http://www.wpdaxue.com/change-password-protected-text.html
 */
function password_protected_change( $content ) {
    global $post;
    if ( ! empty( $post->post_password ) && stripslashes( $_COOKIE['wp-postpass_'.COOKIEHASH] ) != $post->post_password ) {
        $output = '
 
        
<form action="' . get_option( 'siteurl' ) . '/wp-pass.php" method="post"> '.\_\_( "这是一篇受密码保护的文章，您需要提供访问密码：" ).' <label for="post_password">密码：</label> <input class="input" name="post_password" size="20" type="password"></input><input .="" class="button" name="Submit" type="submit" value="' . __( "></input></form> '; return $output; } else { return $content; } } add\_filter( 'the\_content','password\_protected\_change' );
 
```
