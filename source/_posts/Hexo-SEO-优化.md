---
title: Hexo SEO 优化
tags: Hexo
categories: Tools
abbrlink: '1e319277'
date: 2018-08-27 16:45:18
keywords: Hexo SEO 优化 搜索排名 sitemap
description: Hexo SEO优化方法，Google sitemap上传
---

### 添加自己的网站到搜索引擎
#### 生成Sitemap  
安装sitemap插件
```
$ npm install hexo-generator-sitemap --save
$ npm install hexo-generator-baidu-sitemap --save
```
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
<!-- more -->
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
![验证通过之后](\images\Hexo-SEO-优化\1.png)  

### Hexo优化 
#### next开启seo优化
在主题配置文件（`themes/_config.yml`）中修改
```
# seo: false
seo: true
```
开启的时候会导致站点配置文件的description配置失效，此时Sidebar的签名将由`signature`代替([Github Issue][381d0f36])
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
#### 给非友情链接的出站链接添加`nofollow`标签
修改`themes/next/layout/_partials/footer.swig`
```swig
{# 修改此处,"{# #}"是swig中的注释语法 #}
{# {{ __('footer.powered', '<a class="theme-link" target="_blank" href="https://hexo.io">Hexo</a>') }} #}
{{ __('footer.powered', '<a class="theme-link" href="http://hexo.io" rel="external nofollow">Hexo</a>') }}

...

{# <a class="theme-link" target="_blank" href="https://github.com/iissnan/hexo-theme-next"> #}
<a class="theme-link" href="https://github.com/iissnan/hexo-theme-next" rel="external nofollow">
```
修改`themes/next/layout/_macro/sidebar.swig`  
```swig
{# <a href="https://creativecommons.org/{% if theme.creative_commons === 'zero' %}publicdomain/zero/1.0{% else %}licenses/{{ theme.creative_commons }}/4.0{% endif %}/" class="cc-opacity" target="_blank"> #}
<a href="https://creativecommons.org/{% if theme.creative_commons === 'zero' %}publicdomain/zero/1.0{% else %}licenses/{{ theme.creative_commons }}/4.0{% endif %}/" class="cc-opacity" target="_blank" rel="external nofollow">

...

{# <a href="{{ link }}" title="{{ name }}" target="_blank">{{ name }}</a> #}
<a href="{{ link }}" target="_blank" rel="external nofollow">{{ name }}</a>

```
参考链接 [www.arao.me][2dc15000]

  [2dc15000]: http://www.arao.me/2015/hexo-next-theme-optimize-seo/ "Hexo SEO 优化"
  [381d0f36]: https://github.com/iissnan/hexo-theme-next/issues/1484#issuecomment-284316969 "不显示 description"
