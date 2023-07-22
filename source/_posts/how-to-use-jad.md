---
title: class file jad in Linux
excerpt: 
date: 2023-02-27 19:07
categories:
- Java
tags:
- Linux
- jad
- 反编译
---

## 背景
修复工单，本地调试OK，初步排查是线上版本部署代码不是最新的，登录docker后，找到jar文件并解压得到class文件

## Show Code

``` shell
    wget https://www.benf.org/other/cfr/cfr-0.144.jar

    java -jar cfr-0.144.jar your_class.class > out.java
```
