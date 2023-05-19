---
title: "Android mipmap与drawable区别"
date: 2023-05-18T09:29:50+08:00
author: ["otifik"] #作者
draft: false
description: "" #描述
showToc: true # 显示目录
TocOpen: true # 自动展开目录
disableShare: true # 底部不显示分享栏
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
cover:
    image: "" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false
tags: 
- android
---

区别
在apk安装的时候，mipmap-xxx/下的所有分辨率的图片都会保留，而drawablexxx/下的图片只有保留适配设备分辨率的图片，其余图片会丢弃掉，减少了APP安装大小。


> It’s best practice to place your app icons in mipmap- folders (not the drawable-folders) because they are used at resolutions different from the device’s current density。——from Android Developers Blog
>
>Using a mipmap as the source for your bitmap or drawable is a simple way to provide a quality image and various image scales, which can be particularly useful if you expect your image to be scaled during an animation. ——from android-4.3
>
>if you are building different versions of your app for different densities, you should know about the “mipmap” resource directory. This is exactly like “drawable” resources, except it does not participate in density stripping when creating the different apk targets. ——from Google+ Message

用法
mipmapxxx/下放置APP启动图标，以及需要高质量缩放动画的图片。
其他图片资源放在drawablexxx/下
