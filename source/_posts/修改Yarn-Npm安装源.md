---
title: 修改Yarn Npm安装源 以及一些常用命令
tags:
  - npm
  - yarn
categories: 包管理工具
abbrlink: 40bdf958
date: 2019-06-28 11:50:29
---

[Yarn和Npm对比](https://juejin.im/post/5ab89cc4f265da237506e367)
[Yarn安装文档](https://yarnpkg.com/zh-Hans/docs/install#windows-stable)

测试发现虽然npm在升级，但是yarn安装包的速度依然比npm快（2019年6月28日11:57:02）

```javascript
npm config get registry  // 查看npm当前镜像源
npm config set registry https://registry.npm.taobao.org/  // 设置npm镜像源为淘宝镜像

yarn config get registry  // 查看yarn当前镜像源
yarn config set registry https://registry.npm.taobao.org/  // 设置yarn镜像源为淘宝镜像
```

yarn常用命令记录
```javascript
yarn init  //初始化新项目

yarn OR yarn install //安装项目的全部依赖

yarn add [package] //添加依赖包

//将依赖项添加到不同依赖项类别
yarn add [package] --dev
yarn add [package] --peer
yarn add [package] --optional

yarn upgrade [package] //升级依赖包

yarn remove [package] 移除依赖包

yarn global // 在你的操作系统上全局安装包。
可用于 add、bin、list 和 remove 等命令 

yarn global bin 将输出 Yarn 为您已安装的可执行文件之符号链接准备的位置
yarn config set prefix <filepath> 配置此基本位置
```