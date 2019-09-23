---
title: Hexo博客搭建与美化
date: 2019-09-10 17:05:07
tags:
- 博客
- hexo
---

本美博主是在github上搭建了一个免费的博客啦，没有配置域名什么的，但是很好玩～
具体搭建和美化过程如下
<!--more-->
# 搭建
## 创建github仓库
仓库名称：
``` bash
username.github.io
```
其中，username是自己的github用户名，参考[我的github](https://github.com/yihuan18/yihuan18.github.io)

## 安装hexo
``` bash
$ npm install -g hexo
```
## 初始化
首先创建一个hexo博客的根目录
``` bash
$ mkdir hexo
```
然后在目录中进行初始化
``` bash
$ cd hexo
$ hexo init
```
hexo会下载相关内容到目录中
## 构建
然后执行
``` bash
$ hexo g # 生成
$ hexo s (-p port)# 启动服务
```
然后就可以在本地 http://localhost:4000 进行访问，端口号默认为4000
## 上传到github
本地可以正常启动之后，我们可以将博客上传到我们的github中
首先，在_config.yml中配置
```bash
deploy:
  type: git
  repository: git@github.com:username/username.github.io.git
  branch: master
```
然后执行
``` bash
$ hexo d
```
即可
这个时候，我们就可以通过 https://username.github.io/ 访问到自己搭建的博客啦

# 美化
## 主题配置
本美博主当然要换一个小仙女主题啦，个人使用的是[yilia](https://github.com/litten/hexo-theme-yilia)主题，就是你现在看到的博客的亚子
首先需要下载这个主题，注意放到hexo根目录下的themes目录里面哦
``` bash
$ cd ~/hexo/
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```
然后修改_config.yml中的theme
``` bash
theme: yilia
```
重新部署一下就生效啦
``` bash
$ hexo clean
$ hexo g
$ hexo d
```
## 头像修改
将头像图片放在
```
hexo/themes/yilia/source/img
```
(其他的图片都可以放在这个路径下哦)
然后设置主题中的_config.yml配置文件，注意，yilia主题中也有一个同名的配置文件哦
``` bash
avatar: /img/head.png
```
其他的一些美化都可以参考主题作者的[github](https://github.com/litten/hexo-theme-yilia)
