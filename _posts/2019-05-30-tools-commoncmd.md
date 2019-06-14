---
title: 常见的一些命令行
tags: 工具
layout: post
---


总结一些常用的命令。


## Windows

##### 1. 回到跟目录：cd\
##### 2. 到上一层目录：cd..
##### 3. 到其他硬盘(比如D)：d:
##### 4. 查看当前目录下面所有的目录和文件： dir
##### 5. 创建新目录： mkdir 或者 md folder

## Terminal

##### 1. 删除文件夹：rm -rf folderName 或者 sudo rm -rf folderName
##### 2. 查看node装在哪里： ls -la $(which node)

## GIT

##### 1. 克隆和下载项目： git clone  https://github.com/LiMeii/angular-seed-project.git
##### 2. 提交代码： git push -u origin master / git push
##### 3. 设置用户名密码： git config --global user.name "Your Name"  git config --global user.email you@example.com
##### 4. SSL Certificate problem: unable to get local issuer certificate： 解决方案 git config --global http.sslVerify false
    



