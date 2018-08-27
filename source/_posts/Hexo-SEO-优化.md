---
title: Hexo SEO 优化
tags: Hexo
categories: Tools
abbrlink: '1e319277'
date: 2018-08-27 16:45:18
---

### 添加自己的网站到搜索引擎
#### 生成Sitemap  
安装sitemap插件
```
$ npm install hexo-generator-sitemap --save
$ npm install hexo-generator-baidu-sitemap --save
```
<!-- more -->
在站点目录下的_config.yml添加以下内容  
```
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
# url: http://yoursite 此处需要换成自己的域名
url: http://sleepym09.com
root: /
#hexo sitemap
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml
```
在source文件夹下添加robots.txt  
```
User-agent: *
Allow: /
Allow: /archives/
Allow: /categories/
Allow: /tags/ 
Allow: /resources/ 
Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/

// 此处需要替换为自己的域名并删除该行
Sitemap: http://sleepym09.com/sitemap.xml
Sitemap: http://sleepym09.com/baidusitemap.xml
```
然后执行`hexo g`会在public中生成 **sitemap.xml** 和**baidusitemap.xml**，将生成的sitemap.xml上传到**抓取/站点地图**  
#### 站长平台添加自己的站点  
[Google Search Console][b9d0c5c3]  
[百度站长平台][730d921f]

  [b9d0c5c3]: https://search.google.com/search-console "Google站长工具"
  [730d921f]: https://ziyuan.baidu.com/site/siteadd "百度站长平台"
根据各平台的规则添加站点资源，这里以Google为例 
![Google添加站点](\images\Hexo-SEO-优化\1.png)  
验证通过之后
### Hexo优化 
#### 添加关键字  
修改模板`scaffolds\post.md`，添加 keywords 和 description 字段  
```
title: {{ title }}
date: {{ date }}
tags: 
keywords:
description:
```
#### 优化文章链接
安装hexo-abbrlink
```
$ npm install hexo-abbrlink --save
```
修改Hexo默认文章链接Url,编辑站点_config.yml文件，修改permalink字段
```
# permalink: :year/:month/:day/:title/
# permalink_defaults:
permalink: archives/:abbrlink.html
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32
  rep: hex    # 进制：dec(default) and hex
```
