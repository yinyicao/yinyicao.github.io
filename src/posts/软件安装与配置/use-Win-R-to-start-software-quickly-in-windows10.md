---
title: 在Win10中使用Win+R快速启动软件
date: 2021-10-25 10:15:33
tags:
  - 使用技巧
  - windows
categories: 软件安装&配置
---

## 前言

在Win10中，安装某些软件后，我们可以通过`Win+R`按键（以下简称“运行”）直接输入软件名称即可快速启动软件，比如：chrome和firefox。这其实是在软件安装时向注册表`App Paths`中添加了一条记录实现的，如果软件安装时未加入或者加入的名称太长，我们使用运行启动就不好使了。本文记录如何通过修改注册表、添加系统环境变量和使用Cortana来实现通过简单的软件名称简写快速启动。

## 配置环境变量实现(推荐)

1. 新建文件夹shortcut
2. 将需要启动的软件快捷方式复制或新建到shortcut文件夹中（如：IntelliJ IDEA 2020.2.3 x64）
3. 修改快捷方式名称为我们便于使用和记忆的名称（如：idea）
4. 将文件夹路径加入到系统环境变量/用户环境变量中（如：D:\shortcut）
5. 运行（`Win+R`）中输入我们修改后的快捷方式名称（如：idea）即可启动。

## 配置注册表实现

1. 运行（`Win+R`）中输入`regedit`打开注册表
2. 找到`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths\`，此时可以看到我们安装软件默认加入的一些软件路径注册表信息，一个软件一个文件夹(xxxx.exe)
3. 照猫画虎：查看其它软件默认的注册表是怎样的。发现我们只需要在运行中输入注册表中文件夹的前缀就可以直接打开软件（如：xxxx）
4. 在`App Paths`中新建属于某个软件的文件夹（如：idea.exe），修改文件夹中（默认）名称项的数据为软件的具体安装路径（如：`D:\Program Files\JetBrains\IntelliJ IDEA 2020.2.3\bin\idea64.exe`）
5. 运行（`Win+R`）中输入我们在注册表中新建的文件夹名称前缀（如：idea）即可启动。

## Cortana启动

1. 使用Win10自带的Cortana智能助手搜索中直接搜索软件名称回车后即可启动。

## 为何推荐第一种方式

1. 不破坏注册表：注册表中常常存有一些系统关键变量和软件变量，一般正确修改是没有问题，以防万一修改错误造成系统或软件无法运行；
2. 方便：只需要配置环境变量就可以实现，个人觉得比较方便；
3. Cortana一般只能搜索到安装时将快捷方式加入到开始菜单后才可以搜到，一些绿色软件（不用安装的）无法搜到或者需要完整名称才可完全匹配到，效率较低。

## *参考：*

1. [win + R 搜索路径 之注册表 - 小小高 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gaocong/p/15271724.html)
2. [如何使用WIN+R快速启动程序_一苇以航-CSDN博客_win+r打开设置](https://blog.csdn.net/bat67/article/details/76396321)

