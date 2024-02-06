---
title: 发布第一个NPM包 git-push-console
description: 一键推送本地的改动到远端，简化git操作。
tags: npm
categories: Tools
abbrlink: da546d8a
date: 2019-07-23 17:11:23
---
环境要求: NodeJs环境 [NodeJs安装包下载](https://nodejs.org/zh-cn/download/)

这个包简化了git操作命令，可以一键推送本地的改动到远端。中间可以选择输入这次提交的信息，或者使用默认的提交信息。

包的安装使用
```javascript
//安装
npm install git-push-console -g
//用法
命令行进入到项目目录 输入 git-push-console即可。
```

---
### 下面是发布NPM包流程总结
- 新建项目文件夹
- npm init 初始化项目文件夹
```javascript
{
  "name": "git-push-console",
  "version": "1.1.1",
  "description": "",
  "main": "index.js",
  "bin": {
    "git-push-console": "./index.js"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+ssh://git@github.com/yoodz/git-push-console.git"
  },
  "author": "zhengdayong",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/yoodz/git-push-console/issues"
  },
  "homepage": "https://github.com/yoodz/git-push-console#readme",
  "dependencies": {
    "readline-sync": "^1.4.9"
  },
  "keywords": [
    "git",
    "git-push"
  ]
}

```

这里注意下面的配置，全局安装时暴露出操作命令
```javascript
"bin": {
    "git-push-console": "./index.js"
  }
```

- 编写项目代码

新建index.js 文件，编写代码。因为我的文件是在命令行运行的。所以在文件的第一行要输入
```javascript
#!/usr/bin/env node
```


- 注册npm账户
```javascript
https://www.npmjs.com/signup
```


- 命令行登陆npm账户
```javascript
//如果本地修改了npm的安装源，那么需要指定registry, 下同
npm login --registry http://registry.npmjs.org
```


- 推送代码到npm仓库
```javascript
npm publish --registry http://registry.npmjs.org
```

