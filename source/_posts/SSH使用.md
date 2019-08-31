---
title: SSR使用
categories: Tools
tags: Tools
keywords: 多平台SSR使用
abbrlink: 832b0cba
date: 2018-11-15 00:16:23
description:
---
### Windows平台   
打开Git Bash，根据规则输入下面语句
```
# -f 生成的RSA密钥对名称
# -t type 认定方式
# -C 公钥中的备注信息
ssh-keygen -f fileName -t rsa -C "YourEmail@example.com"
```
![](\images\SSH使用\1.png)  
最终在当前路径下生成`fileName`(私钥)、`fileName_pub`(公钥)
