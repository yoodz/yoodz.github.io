---
title: 用了React或者Vue单页应用，如何做SEO优化呢？
abbrlink: 795ce7bb
description: 其实这个标题更贴切的意思是说，如果你已经用了React做了网站，该怎么做SEO呢？因为如果你还没开始做网站，并且该网站很注重SEO，我会推荐你直接放弃做成单页应用。因为vue也可以只像JQuery一样，做一些交互增强。
date: 2020-03-28 16:23:15
tags: 
  - 技术
  - SEO
---
其实这个标题更贴切的意思是说，如果你已经用了React做了网站，该怎么做SEO呢？因为如果你还没开始做网站，并且该网站很注重SEO，我会推荐你直接放弃做成单页应用。因为vue也可以只像JQuery一样，做一些交互增强。
事情起因，这段时间一些前端的框架比较火，我也看了一些React和Vue的文档，刚好公司有一个新的官网需要开发，我就想正好边学边用，于是用上了React。谁知道官网上线之后老板一直盯着网站的SEO，说在百度怎么都搜索不到网站。于是我开始了一系列的SEO优化。因为网站关键字不是实时收录的，每次更改之后要几天之后才能看到效果，所以这个过程持续了比较久的时间，还是比较曲折的。
百度收录网站时如果遇到这种单页应用，就只会收录入口文件，也就是那个index.html，因为里面的内容都是通过js动态注入进去的，所以爬虫抓不到页面的其他内容，这样你的关键字就会很少，别人搜索时命中的机会就会很少。
我优化这个项目主要从以下几个方面：
- 优化index.html里的标题，关键字，描述
- 去掉访问路径里的#
- 尝试预渲染，SSR
- 新增静态页面


我的项目是通过React的脚手架工具(create-react-app)直接生成的.
```
#生成的项目结构大致是这样

my-app
├── README.md
├── node_modules
├── package.json
├── .gitignore
├── config
│   ├── webpack.config.dev.js
│   ├── webpack.config.prod.js
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── manifest.json
│   └── robots.txt
└── src
    ├── App.css
    ├── App.js
    ├── App.test.js
    ├── index.css
    ├── index.js
    ├── logo.svg
    └── serviceWorker.js
```


### 优化index.html里的标题，关键字，描述
目前项目里唯一一个能够被爬虫爬到的页面，当然要好好发挥了。
![修改文件标题](https://static.afunny.top/2023/202304200926741.png)

### 去掉访问路径里的 井号
把默认的HashRouter 改为 BrowserRouter，就去掉了访问路径里的井号，亲测可用。
![修改路由模式](https://static.afunny.top/2023/202304200926171.png)
yarn build完成之后，部署会产生404错误。通过配置nginx解决页面404错误，只需要访问任何路由地址都访问index.html，这样就可以自动被React-Router处理，并进行无刷新跳转。我们在nginx服务器的location中添加代码行 try_files $uri $uri/ /index.html 即可，部分配合代码如下：
```
# nginx 的修改如下

server
{
    listen 80 default_server;
    listen [::]:80 default_server;

    root /usr/local/react/build;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html;

    server_name example/test; 

    location / {
        try_files $uri $uri/ /index.html;   //主要添加这一行
    }
}
```


### 尝试预渲染，SSR
这是在搜索怎么解决SEO问题时，在为数不多的信息中找到的说的比较多的方式。
#### 预渲染
```
# 首先安装需要的插件
npm install prerender-spa-plugin --save



#配置webpack生成规则，修改文件位置 config/webpack.config.prod.js。
# 首先引入插件
const PrerenderSPAPlugin = require('prerender-spa-plugin');



# plugins 中初始化插件，其中的router是项目的路由
new PrerenderSPAPlugin({
    routes: ['/', '/gameDetail'],
    staticDir: path.join(__dirname, '../build'),
})
```
配置完成之后，再次执行yarn build，会发现编译出来的代码，对应给每个路由建立文件夹。这个放在服务器上几天感觉并没有效果，后来就没用了。

#### SSR
SSR的教程网上也一搜一大把，主要的思想是服务端渲染，使用Node在服务端，使用renderToString 生成html代码，去替换掉 index.html 中的 {{root}} 部分。我看了教程需要处理的问题太多，感觉需要的时间都够我重新写一遍网站了。而且我现在服务器静态网站都是用Nginx部署，再加个Node怕是还要做转发。如果只是为了SEO，真是得不偿失。所以我只是尝试了示例代码，放弃这种了方式。
因为我还想到了另外一个骚操作。

### 新增静态页面
查看上面的项目结构可以发现，public文件夹下的所有文件都会被编译到最终的输出文件里，那我是不是可以在public下面加一些静态页面，来做SEO?
![添加静态页面后的public文件夹](https://static.afunny.top/2023/202304200926192.png)
这些静态页面是直接从单页网站里复制出来的，具体操作方式是，打开单页网站，右键查看网页源代码，复制所有代码为HTML文件。
然后需要修改的两个地方。
- 固定里面的css，原来的css是生成的不规则文件名，这里要固定一个。如上面的base.css。
- 删除里面引入的JS。

先做这些优化，如果还不理想，后期多增加一些静态页面再看看效果。当然普遍的SEO方式还是可以同时进行的，比如内链、外链、标题字数、标题分割符、运用一些分析网站检查等等。

### 附上VUE作者对SEO问题的解答。
```
我就说一点，用 Vue 不代表你一定要做成 SPA。现在有些人提起 Vue 就好像一定要 CLI 全家桶，
其实 Vue 从一开始就一直很注重对后端渲染的应用做渐进增强的用例，现在欧美也有很多的开发者拿
Vue 直接替代 jQuery 做常见的交互增强。对于真正适合做成 SPA 的应用，SEO 反而通常不是问题。
你针对 marketing 的页面应该是静态分开部署的，app 本身则要登陆才能用，SEO 没有什么意义。
少数既需要 SPA 强交互性，又对 SEO 和首屏速度有刚性需求的场景，这时候同构 SSR 就派上用场了。

作者：尤雨溪
链接：https://www.zhihu.com/question/51949678/answer/146656850
```