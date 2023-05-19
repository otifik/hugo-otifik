---
title: "Scoop的安装使用"
date: 2023-03-14T19:34:52+08:00
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
- Scoop
---

# Scoop

windows包管理器

## 安装

管理员模式打开PowerShell，设置用户安装路径和全局安装路径，

```powershell
$env:SCOOP='D:\Scoop'
[Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')
$env:SCOOP_GLOBAL='D:\ScoopGlobal'
[Environment]::SetEnvironmentVariable('SCOOP_GLOBAL', $env:SCOOP_GLOBAL, 'Machine')
```

设置PowerShell执行策略为RemoteSigned

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

安装Scoop

```powershell
iwr -useb get.scoop.sh | iex
```

Scoop中安装git

```powershell
scoop install git
```

**最好打开git初始化一下，否则会出现timeout等情况**

## 使用

### 添加bucket

```powershell
scoop bucket add <bucketname>
```

### 安装软件

```powershell
scoop install <app>
```

### 管理软件缓存

```powershell
scoop cache show 	 	#显示安装包缓存
scoop cache rm <app> 	#删除指定应用的安装包缓存
scoop cache rm * 	 	#删除所有的安装包缓存
```

### 全局安装软件

```powershell
scoop install -g <app>
```

### Aria2多线程下载

```powershell
scoop install aria2
```

配置相关设置

```powershell
scoop config aria2-split 32#单任务最大连接数设置为32
scoop config aria2-max-connection-per-server 16#单服务器最大连接数设置为16
scoop config aria2-min-split-size 1M#最小文件分片大小设置为1M
```

### 常用命令

```powershell
# 更新 scoop 及软件包列表
scoop update

## 安装软件 ##
# 非全局安装（并禁止安装包缓存）
scoop install -k <app>
# 全局安装（并禁止安装包缓存）
sudo scoop install -gk <app>

## 卸载软件 ##
# 卸载非全局软件（并删除配置文件）
scoop uninstall -p <app>
# 卸载全局软件（并删除配置文件）
sudo scoop uninstall -gp <app>

## 更新软件 ##
# 更新所有非全局软件（并禁止安装包缓存）
scoop update -k *
# 更新所有软件（并禁止安装包缓存）
sudo scoop update -gk *

## 垃圾清理 ##
# 删除所有旧版本非全局软件（并删除软件包缓存）
scoop cleanup -k *
# 删除所有旧版本软件（并删除软件包缓存）
sudo scoop cleanup -gk *
# 清除软件包缓存
scoop cache rm *
```

