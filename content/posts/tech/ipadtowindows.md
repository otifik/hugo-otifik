---
title: "iPad文件传输到windows"
date: 2023-05-03T18:04:56+08:00
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
- ios
- windows
---

# iPad文件传输到windows

处于同一wifi下的iPad与windows文件传输，务必阅读完再进行操作，亲测13g视频花了大概十分钟不到

## 准备工作：打开SMB

**控制面板**-**程序**-**启用或关闭windows功能**

![](image-20230503171053590.png)

确定后重启电脑

## 创建共享文件夹

任意位置新建文件夹，右键**属性**-**共享**

![image-20230503171606431](image-20230503171606431.png)

点击共享

![image-20230503171820453](image-20230503171820453.png)

选择自己的账号并**添加**，然后点击**共享**（一般都默认会有自己的账号，没有的情况请自行创建一个本地账号）

![image-20230503171843029](modified.png)



## iPad连接

然后iPad上打开文件，点击**三个点图标**-**连接服务器**

![img](80601909ABC7EC8078C9A4ADD6C7C267.png)

输入smb://+电脑端的ip地址（cmd通过ipconfig获取，不多赘述）

![img](7802F3DCEB2A6FFF39F02BC22F43F0DD.png)

选择注册用户，输入windows用户名和密码

![img](728E3B0C6DC227306D0FA81D45B417DE.png)

用户就是C:/Users下的用户，密码就是pin

![image-20230503173410051](image-20230503173410051.png)

## 上传文件至共享文件夹

点击**分享**-**存储到文件**，然后直接点共享文件夹存进去就好啦

![img](3DE6B68E5174331E62012EB02F48682D.png)

![img](D21AC5D2C4CB21B695F87BC4FB9002C2.png)

![img](4BBD7A2D3011C30C8E0FD71011BBE998.png)