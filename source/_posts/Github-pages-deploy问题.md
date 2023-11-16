---
title: Github pages deploy问题
date: 2023-11-16 17:36:15
tags: debug
---

## 描述
利用GitHub Action部署源码在master分支上的hexo框架页面时发现deploy步骤运行失败并报错：
```
error: Branch "master" is not allowed to deploy to github-pages due to environment protection rules.
error: The deployment was rejected or didn't satisfy other protection rules.
```

## 原因
由于从`Deploy from branch`切换到`Github Action`，生成页面的branch从`gh-pages`变为`master`，而原本的部署自动生成了名为github-pages的环境，并自动应用了对gh-pages分支的保护。当将source branch切换为master后，原保护规则并未自动修改，导致deploy因master分支无保护而失败。

## 解决
Settings > Environment > 删除github-pages环境

## 参考
https://github.com/orgs/community/discussions/39054