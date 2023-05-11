---
title: github+picgo+jsdelvr搭建免费图床
comments: true
categories:
  - default
tags:
  - default
aside: true
highlight_shrink: true
abbrlink: dbed16f2
date: 2022-10-14 23:00:33
updated:
description:
keywords:
cover: /image/jsdelivr_github.png
---

如果我们平时写博客用的是markdown来进行编写，肯定会遇到图片的存储问题。如果放在本地，文章发送给别人看，图片将无法解析出来，这时候就需要用到图床——一种在云端上存储图片的服务器，提供链接供我们访问图片，我们只需要在文章中插入对应链接即可实现一次储存，多地使用。本文将交大家如何利用github+picGo快速搭建免费图床，并使用jsdelivr免费做CDN加速。

## github准备

1. 准备一个图床仓库

   ![新建图床仓库](https://cdn.jsdelivr.net/gh/Li-Changwu/image/default/20221014230729.png)

   > 很多博客教学让克隆Github仓库到本地和发布仓库Release，其实如果仅仅作为图床使用可以不用做这几步操作。

2. 获取github的Token授权，后面picgo需要用到

   在github->setting->Developer settings的Personal access tokens中点击Generate new token生成新的Token
   ![token位置](https://cdn.jsdelivr.net/gh/Li-Changwu/image/default/20221014231554.png)
   填写Token相关信息，Note是Token的名字，随意就好，Expiration是Token过期时间，建议设置为No expiration，也就是永不过期，权限勾选下图中这几个就行，也可以根据自己情况勾选，然后点击最下方的Generate Token。
   ![生成新token](https://cdn.jsdelivr.net/gh/Li-Changwu/image/default/20221014232012.png)
   界面这个位置会显示生成的Token，请复制保存好，它只会显示一次。
   ![Token](https://cdn.jsdelivr.net/gh/Li-Changwu/image/default/20221014232550.png)

## Picgo准备

1. 去[picgo GitHub仓库](https://github.com/Molunerfinn/PicGo/releases)下载对应版本的安装包安装

2. 打开Picgo软件，点击图床配置，选择GitHub图床，填写相关配置

   ![picgo配置](https://cdn.jsdelivr.net/gh/Li-Changwu/image/default/20221014233301.png)

+ 设定仓库名：你的`github名/仓库名`
+ 设定分支名：main
+ 设定Token：前面获取的Github Token
+ 指定存储路径：相当于在仓库里的文件夹路径
+ 设定自定义域名：我们采用jsdelivr作CDN加速，因此这里填写的域名根据jsdelivr的规则应该是`https://cdn.jsdelivr.net/gh/你的GitHub名/你的仓库名`，详见[jsdelivr官网](https://www.jsdelivr.com/)。

欧克，到此，你已经拥有了一个属于你自己的图床，点击上传区即可上传图片到图床，picGo会返回对应的图片链接，并且提供多种链接格式可以选择。

![image-20221015095706548](https://cdn.jsdelivr.net/gh/Li-Changwu/image/default/20221015111143.png)

## Typaro（一种简介方便的markdown编辑器）

如果你使用Typaro进行文章编写，可以在文件->偏好设置->图像里面配置PicGo进行图片上传，只需在填入picGo对应路径即可。现在你就可以直接使用CV大法直接插入图片了，图片会自动上传图床。

![image-20221015095952444](https://cdn.jsdelivr.net/gh/Li-Changwu/images/default/image-20221015095952444.png)
