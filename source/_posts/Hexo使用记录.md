---
title: Hexo使用记录
tags: Hexo
categories: Tools
abbrlink: '472738e0'
date: 2018-08-23 16:35:55
keywords: Hexo 个性化设置 
description: Hexo使用记录、个性化设置
---

- <a href="#1" target="_self">1. Hexo安装&部署</a>
- <a href="#2" target="_self">2. Hexo个性化设置</a>
    - <a href="#2.1" target="_self">2.1 开启emoji</a>
    - <a href="#2.2" target="_self">2.2 设置锚点</a>

<!-- more -->
### <span id = "1"><font >1. Hexo安装&部署</font></span>

#### <span id = "1.2"><font >1.2 Github配置</font></span>  
具体配置可以参考 [这篇文章][02a195c0],写的很详细。

### <span id = "2"><font >2. Hexo个性化设置</font></span>
#### <span id = "2.1"><font >2.1 开启emoji</font></span>
[参考这里][9433a787]
  开启emoji  
  $ npm install hexo-filter-github-emojis --save  
 打开站点配置文件 添加以下内容
 ```
 githubEmojis:
  enable: true
  className: github-emoji
  unicode: false
  styles:
  localEmojis: 
 ```
 附带[表情表][aed4a2e5]
#### <span id = "2.2"><font >2.2 设置锚点</font></span>
锚点可以帮我们实现章开头的文章列表跳转效果
在标题处用如下代码设置锚点：

```html
    <span id = "2.2"><font >2.2 设置锚点</font></span>
```
例如我想<a href="#2.2" target="_self">点击示例</a>跳转到***2.2 设置锚点***，则应该用如下代码设置跳转：

```html
    <a href="#2.2" target="_self">点击示例</a>
```
href:填写要跳转锚点处定义的id（锚点处的id可以随便设置，此处我设置为2.2），
target:表示浏览器跳转锚点的不同行为，具体区别可以[参照这里](http://www.w3school.com.cn/tags/att_a_target.asp)   

  [02a195c0]: https://juejin.im/entry/5a574864f265da3e3c6c1217 "Hexo搭建"
  [aed4a2e5]: https://www.webfx.com/tools/emoji-cheat-sheet/ "emoji表"
  [9433a787]: https://novnan.github.io/Hexo/emojis-for-hexo-next/ "Hexo支持emoji"
  [e931a755]: https://git-scm.com/ "Git下载地址"
  [4872c4f3]: https://nodejs.org/zh-cn/ "Node.js"
  [2f255e98]: https://atom.io/ "Atom"
  [8ce4df2e]: http://sleepym09.com/2018/08/24/Atom%E4%BD%BF%E7%94%A8%E8%AE%B0%E5%BD%95/ "Atom使用记录"
