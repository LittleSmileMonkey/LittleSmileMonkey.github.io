---
title: Hexo多客户端同步问题
tags: Hexo
categories: Tools
abbrlink: 9f9d3eee
date: 2018-08-24 11:42:16
---
>在家里同步github上的博客源码时发现无法同步themes/next,本文主要记录用git subtree解决该问题(其他theme方法相同)，对于Git使用不太熟悉的朋友
>可以看下[这篇文章 ][5421a066]

  [5421a066]: http://iissnan.com/progit/ "Pro Git"
### 原因  
next的集成文章中是用clone方法来使用的，而git无法直接管理这样的嵌套模块。  
<!-- more -->
### 解决
######  方案一： 简单粗暴解决 
  直接删掉next/.git文件夹，将next当成普通的文件加入到版本控制当中，但是这样操作之后无法更新next主题。
######  方案二： subtree
1. 首先需要在自己的github上fork一份next源码  
  ![fork源码](https://user-gold-cdn.xitu.io/2018/8/27/1657a33d797b3d51?w=920&h=246&f=png&s=22592)  
  接着会在自己的github Repositories中多出一个同名项目    
  ![自己仓库](https://user-gold-cdn.xitu.io/2018/8/27/1657a34c2682fb3d?w=976&h=190&f=png&s=13651) 

2. 添加subtree  
  ``` 
  # 添加远程仓库 
  # git remote add -f <子仓库名> <子仓库地址>
  $ git remote add -f next git@github.com:LittleSmileMonkey/hexo-theme-next.git
  
  # git subtree add --prefix=<子目录名> <子仓库名> <分支> --squash
  $ $ git subtree add --prefix=themes/next next master --squash
  ```
  此阶段碰到的bug:
  因为之前已经clone过next项目到themes/next中，所以直接操作第三步第二条命令时会出现下面错误  
  ![subtree add 报错](https://user-gold-cdn.xitu.io/2018/8/27/1657a3569761775f?w=468&h=65&f=png&s=5589)  
  尝试删除next文件夹然后执行（记得将自己修改过的例如_config.yml备份），会出现下面错误  
  ![删除next后报错](https://user-gold-cdn.xitu.io/2018/8/27/1657a35d8c59ed9a?w=453&h=52&f=png&s=5521)  
  删除本地仓库的themes/next 并提交到远程仓库以保证working tree处于未修改状态
  ![完成](https://user-gold-cdn.xitu.io/2018/8/27/1657a361cab89ed2?w=468&h=86&f=png&s=6193)
  至此已将自己fork的next主题当成subtree添加到项目中  
3. 推送next中的修改  
  其实对整个项目的pull、push同样会对子项目起作用(比较推荐)，这里记录一下单独对next子项目进行pull、push操作  
  我们将目录中的next/_config.yml替换为之前备份好的，然后执行下面命令推送到远程仓库
  ```
  # 将修改的文件commit到本地仓库
  $ git commit -a -m '更新备份的_config.yml文件'
  
  # git subtree push --prefix=<子目录名> <远程分支名> 分支
  $ git subtree push --prefix=themes/next next master  
  
  # git subtree pull --prefix=<子目录名> <远程分支名> 分支
  $ git subtree pull --prefix=themes/yilia yilia master  --squash
  ```
  此时可以看到fork的next仓库更新了  
  ![更新完成](https://user-gold-cdn.xitu.io/2018/8/27/1657a366035fb854?w=1131&h=50&f=png&s=5383) 
   
######  方案三： [submodule][76531f62] 
没有实践过，但是从网上subtree和submodule的对比来看，似乎subtree完胜submodule,因此如果有愿意踩坑的小伙伴可以自行尝试。

[76531f62]: https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97 "submodule"
