---
title: Hexo多客户端同步问题
date: 2018-08-24 11:42:16
tags: Hexo
categories: 开发工具
---
>在家里同步github上的博客源码时发现无法同步themes/next,本文主要记录用git subtree解决该问题(其他theme方法相同)，对于Git使用不太熟悉的朋友>可以看下[这篇文章 ][5421a066]

  [5421a066]: http://iissnan.com/progit/ "Pro Git"
### 原因  
是因为next的集成文章中是用clone方法来使用的，而git无法直接管理这样的嵌套模块。  
### 解决
######  方案一： 简单粗暴解决方法 
  直接删掉next/.git文件夹，将next当成普通的文件加入到版本控制当中，但是这样操作之后无法更新next主题的更新。
######  方案二： subtree解决方法
1. 首先需要在自己的github上fork一份next源码  
  ![fork源码](\images\Hexo多客户端同步问题\1.png)  
  接着会在自己的github Repositories中多出一个同名项目    
  ![自己仓库](\images\Hexo多客户端同步问题\2.png)  

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
  ![  subtree add 报错](\images\Hexo多客户端同步问题\1.png)  
  尝试删除next文件夹然后执行（记得将自己修改过的例如_config.yml备份），会出现下面错误  
  ![  删除next后报错](/images/2018/08/1.png)  
  删除本地仓库的themes/next 并提交到远程仓库以保证working tree处于未修改状态
  ![  完成](/images/2018/08/1.png)
  至此已将自己fork的next主题当成subtree添加到项目中  
3. 推送next中的修改  
  其实对整个项目的pull、push同样会对子项目起作用，这里记录一下单独对next子项目进行pull、push操作  
  我们将目录中的next/_config.yml替换为之前备份好的，然后执行下面命令推送到远程仓库
  ```
  # git subtree push --prefix=<子目录名> <远程分支名> 分支
  $ git subtree push --prefix=themes/next next master  
  ```
