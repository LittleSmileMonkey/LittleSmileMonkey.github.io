---
title: Assets相关
tags: assets
categories: Android
abbrlink: '87276193'
date: 2018-08-27 14:45:33
keywords: 序列化 Assets 指定外部目录
description: Android Assets 使用、指定外部目录为Assets目录
---
### 创建assets文件夹

![在AS中创建assets文件夹](https://user-gold-cdn.xitu.io/2018/7/23/164c6616442c2f81?w=823&h=529&f=png&s=90565)  
<!-- more -->
并且会更改对应module的.iml文件，生成`<sourceFolder url="file://$MODULE_DIR$/src/main/assets" type="java-resource" />`和`<option name="ASSETS_FOLDER_RELATIVE_PATH" value="src/main/assets" />`
，其中file://后面是assets文件的绝对路径。  

有时候会碰到创建了Assets并放入文件，但是打包之后无法读取的情况，这个时候我们可以看看apk结构，看assets是否被打包进Apk，例如：
![APK结构](https://user-gold-cdn.xitu.io/2018/7/23/164c66d9340e854a?w=1419&h=252&f=png&s=30008)
如果在Apk中没有发现assets，则需要检查.iml里的配置是否正确。  

### 从Assets中读取文件  

```
//因为是本剧bf.readLine()是否为空来判断，所以中间不能有空行
    private fun getAssetsText(fileName: String): String {
        val stringBuilder = StringBuilder()
        try {
            val bf = BufferedReader(InputStreamReader(
                    assets.open(fileName)))
            var lines: String
            while (true) {
                //当有内容时读取一行数据，否则退出循环
                lines = bf.readLine() ?: break
                stringBuilder.append(lines) //打印一行数据
            }
        } catch (e: IOException) {
            e.printStackTrace()
        }
        return stringBuilder.toString()
    }
```

### 指定Project外部文件夹作为APK的assets目录，并过滤指定文件(夹)  

有时候会碰到这样的情况：cocos开发中用到的lua代码，每次编译需要手动将部分代码移动到android assets文件夹下，很麻烦，所以需要指定不在android project下的目录作为assets，打包进APK中。  
在build.gradle中有设置assets路径的方法：  

```
android {
    ...
    aaptOptions {
        ignoreAssetsPattern "<dir>nameA:thumbs.db:!.svn"
    }

    sourceSets{
        main {
            assets.srcDirs = ['D://Assets']
        }
    }
    ...
}
```

其中`assets.srcDirs = ['']`可以填写相对路径也可以填写绝对路径，相对的是当前module路径，绝对路径可以指向外部任何一处，多处路径之间用`,`隔开。  
`ignoreAssetsPattern`的配置就是忽略assets目录中符合`""`中填写正则的文件/文件夹**不被打包**到APK中，多个正则之间用`:`隔开。
