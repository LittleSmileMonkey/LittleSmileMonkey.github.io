---
title: Hexo使用记录
date: 2018-08-23 16:35:55
tags: Hexo
---

- <a href="#1" target="_self">1. Hexo安装&部署</a>
    - <a href="#2" target="_self">1.1 环境配置</a>
    - <a href="#2" target="_self">1.2 Github配置</a>
    - <a href="#2" target="_self">1.3 部署</a>
- <a href="#2" target="_self">2. Hexo个性化设置</a>
    - <a href="#2.1" target="_self">2.1 设置头像</a>
    - <a href="#2.2" target="_self">2.2 设置锚点</a>

<!-- more -->
### <span id = "1"><font >1. Hexo安装&部署</font></span>

### <span id = "2"><font >2. Hexo个性化设置</font></span>
#### <span id = "2.1"><font >2.1 Hexo个性化设置</font></span>

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
ps:这里推荐一款mark
