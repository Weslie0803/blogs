---
title: typescript安装问题
date: 2023-11-15 12:22:09
tags:
---

## 描述
使用yarn安装typescript时成功，但运行报错：
```
> tsc -v
tsc: command not found
```

## 原因
yarn路径未添加至环境变量，未找到yarn下载的typescript二进制文件

## 解决
- 运行`yarn global bin`获取yarn二进制下载路径
- 将路径添加至环境变量
```
export PATH=$PATH:/path/to/yarn/bin
```