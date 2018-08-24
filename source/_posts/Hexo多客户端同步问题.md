---
title: Hexo多客户端同步问题
date: 2018-08-24 11:42:16
tags: Hexo
categories: 开发工具
---
>在家里同步github上的博客源码时发现无法同步themes/next,本文记录用git subtree解决该问题(其他theme方法相同)，对于Git使用不太熟悉的朋友>可以看下[这篇文章 ][5421a066]

  [5421a066]: http://iissnan.com/progit/ "Pro Git"
### 原因  
是因为next的集成文章中是用clone方法来使用的，而git无法直接管理这样的嵌套模块
1. 首先需要在自己的github上fork一份next源码  
  ![fork源码](/images/2018/08/1.png)  
  接着会在自己的github Repositories中多出一个同名项目    
  ![自己仓库](/images/2018/08/2.png)  

2. 
