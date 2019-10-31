---
title: ubuntu远程桌面
typora-root-url: ../../source/
date: 2019-10-31 20:04:37
tags:
- ubuntu
- 远程桌面
---

# mac上使用远程桌面连接ubuntu

今天在教研室拆键帽清洁，不小心把一个键帽连轴拔起。。。于是台式没有键盘用了
然后想直接在mac上使用远程桌面使用台式机
台式系统为ubuntu 16.04，我们要使用 vnc 远程桌面 ubunt16.04的自带桌面 unity，教程如下：

## 安装x11vnc
1. 安装
```bash
sudo apt-get install x11vnc
```
2. 设置密码
```bash
x11vnc -storepasswd
```
3. 启动vnc服务
```bash
x11vnc -forever -shared -rfbauth ~/.vnc/passwd
```

## 启动mac远程桌面

1. 打开mac自带的远程桌面
2. 输入{ip:端口号}

![image-20191031201539487](/imgs/image-20191031201539487.png)

3. 输入密码连接即可

效果如图：

![image-20191031201843585](/imgs/image-20191031201843585.png)