---
title: CJS & ESM 模块转换
abbrlink: 840d27f9
date: 2023-08-08 09:10:41
tags: Javascript
# thumbnail: "https://static.afunny.top/2023/202304200848795.png"
---
CJS（Commonjs）使用在后端的，为了适应前端有了AMD（Asynchronous Module Definition），或CMD（Common Module Definition）。AMD 和 CMD 是社区的开发者们制定的模块加载方案，并不是语言层面的标准。**从 ES6 开始，在语言标准的层面上，实现了模块化功能ESM（**ES6 Module**），而且实现得相当简单，完全可以取代 CommonJS 和 CMD、AMD 规范，成为浏览器和服务器通用的模块解决方案**。

# 回顾没有模块的加载

总结有三个问题
- 全局作用域污染
- 数据保护问题
- 模块之间相互依赖问题

## 全局作用域污染

内嵌式和外联式。最初的js都是卸载html文件的script标签里。后来随着逻辑变复杂，开始抽离出单独的文件，采用外联式引入到页面内。

```jsx
// index.html 内嵌
<script>
  var a = 1;
  var b = 2;
</script>

// index.html 外联
<script src="./a.js"></script>
<script src="./b.js"></script>
<script src="./c.js"></script>
```

Javascript在es6之前没有模块系统。所有的变量存在全局作用域中，所以有全局作用域污染的问题。

## 数据保护问题

手动划分模块。模块属性可以随意被修改。比如	app.moduleA.name = 'rename'

```jsx
// index.html
<script>
	app.moduleA = {}
	app.moduleA.name = 'hello world'
	app.moduleB  = {}
	app.moduleB.name = 'hello js'
</script>
```

闭包可以解决全局作用域，和数据保护问题。

# 闭包创建局部作用域

闭包内的属性，通过暴漏的方法，获取和修改属性值。

```jsx
var moduleA = (function() {
    var name = 'zdy-a';
    var age = 23;
    return {
        getName: () => {
			// Uncaught TypeError: Cannot read properties of undefined (reading 'getName')
            console.log(moduleB.getName(), 'moduleDemo-26')
            return name;
        },
        rename: function(newName) {
            name = newName
        }
    }
})()
console.log(moduleA.getName(), 'moduleDemo-30')
moduleA.rename('shayebushi')

var moduleB = (function() {
    var name = 'zdy-b';
    var age = 18;
    return {
        getName: () => {
            return name;
        },
        rename: function(newName) {
            name = newName
        }
    }
})()
```

因为有加载顺序，所有先加载的方法不能够调用后加载方法。有模块间相互依赖的问题。

# 常见的模块加载方式

## 服务端

CJS - Node.js

每个文件是一个模块。每个模块有两个变量，require 和 module 。require 用来加载模块。module代表当前的模块。exports是module上的属性(module.exports = {})，保存了当前模块导出的变量。

```jsx
// lib.js
const name = 'zdy';
const age = 18;

module.exports = {
    name,
    age
}
// 也可以这样导出
// exports.name = name;
// exports.age = age;

// index.js
const lib = require('./lib');

console.log(lib.name, 'index-3') // zdy index-3
console.log(lib.age, 'index-4') // 18 index-4
```

同时为了方便，Node.js 在实现 CommonJS 规范时，为每个模块提供一个 exports的私有变量，指向 module.exports。类似 **var** exports **=** module.exports。所以也可以使用exports导出。

```jsx
// lib.js
const name = 'zdy';
const age = 18;
// 也可以这样导出
exports.name = name;
exports.age = age;
```

require 命令的基本功能是，读入并执行一个 js 文件，然后返回该模块的 exports 对象。如果没有发现指定模块，会报错。另外`require` 的是被导出的值的拷贝。

## 客户端

AMD - Require.js

```jsx
// amd-1.js
define(function() {
  'use strict';
  const name = 'amd1';

  return {
      name: name
  }
});

// amd-1.js, 依赖amd-1.js
define(function(require) {
  'use strict';
  var amd1 = require('./amd-1')
  console.log(amd1.name, 'amd-2-4')
});

// index.html
<script src="https://cdn.jsdelivr.net/npm/requirejs@2.3.6/require.min.js"></script>
<script>
    requirejs(['./amd-2.js']);
</script>

```

CMD - Sea.js

## 客户端&服务端加载

ESM -ES6

ESM的模块加载,两种导出方式，三种导入方式，使用export 和 import加载

```jsx
// 导出方式
const a = 1;
export { a };
export default a;

// 导入方式
import { a } from './xxx';
import a from './xxx';
import * as a from './xxx';
```

# ESM 和 CJS的区别

CJS运行时确认导出的接口，导出的是对象；ESM编译时确定模块依赖。

require导入值的拷贝

```jsx
// requireModule.js
var age = 19;

function changeAge(value) {
    age = value
}

module.exports = {
    age,
    changeAge
}
// main.js
const requireModule = require('./requireModule');

console.log(requireModule.age, 'index-4') // 19
requireModule.changeAge(45)
console.log(requireModule.age, 'index-11') // 19
```

import 导入值的引用

```jsx
// importModule.js
var age = 19;

function changeAge(value) {
    age = value
}

export {
    age,
    changeAge
}

// main.js
import * as importModule from "./importModule.js";

console.log(importModule.age, 'index-4') // 19
importModule.changeAge(89)
console.log(importModule.age, 'index-6') // 89
```

# ****ESM 和 CJS 互相转换****

开发中可能会遇到一些需要相互转换的场景。

- ESM 引入只支持 CJS 的库
- 开发 npm 库的时候，写 ESM 然后编译成 CJS。

## ESM转CJS

- npm库一般是ESM开发，同时提供ESM和CJS。
- 业务开发使用ESM，考虑兼容使用打包工具（webpck）编译成CJS线上运行。
    - ESM规范编写代码，使用**`import`**、**`export`**;
    - babel等编译器将ESM代码转成CJS代码；
    - 浏览器不支持CJS规范，所以webpack按照CJS规范实现了类似**`require`**和**`module.exports`**的模块加载机制。

### export导出

默认导出

```jsx
export deafult 'monday';
转换为
modules.exports = 'monday';
```

命名导出

```jsx
export const a =  123;
export const b = 456;
转换成
module.exports.a = 123
module.exports.b = 234
```

默认导出+命名导出

```jsx
// mixModules.js
export default 666
export const a = 123
export const b = 234

// index.js
import lib from './mixModules'
import {a, b} from './mixModules'

console.log(lib, a, b)
```

上面的代码webpack编译后，删除注释后结果如下：

```jsx
(function () {
  "use strict";
  var __webpack_modules__ = ({
    "./src/index.js": (function (__unused_webpack_module, __webpack_exports__, __webpack_require__) {
      eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _mixModules__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./mixModules */ \"./src/mixModules.js\");\n\n\n\nconsole.log(_mixModules__WEBPACK_IMPORTED_MODULE_0__[\"default\"], _mixModules__WEBPACK_IMPORTED_MODULE_0__.a, _mixModules__WEBPACK_IMPORTED_MODULE_0__.b)\n\n//# sourceURL=webpack://webpack/./src/index.js?");
    }),

    "./src/mixModules.js": (function (__unused_webpack_module, __webpack_exports__, __webpack_require__) {
      eval("__webpack_require__.r(__webpack_exports__);\n/* harmony export */ __webpack_require__.d(__webpack_exports__, {\n/* harmony export */   a: function() { return /* binding */ a; },\n/* harmony export */   b: function() { return /* binding */ b; }\n/* harmony export */ });\n/* harmony default export */ __webpack_exports__[\"default\"] = (666);\nconst a = 123\nconst b = 234\n\n//# sourceURL=webpack://webpack/./src/mixModules.js?");
    })

  });
  var __webpack_module_cache__ = {};
  function __webpack_require__(moduleId) {
    var cachedModule = __webpack_module_cache__[moduleId];
    if (cachedModule !== undefined) {
      return cachedModule.exports;
    }
    var module = __webpack_module_cache__[moduleId] = {
      exports: {}
    };

    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);
    return module.exports;
  }

  !function () {
    __webpack_require__.d = function (exports, definition) {
      for (var key in definition) {
        if (__webpack_require__.o(definition, key) && !__webpack_require__.o(exports, key)) {
          Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });
        }
      }
    };
  }();

  !function () {
    __webpack_require__.o = function (obj, prop) { return Object.prototype.hasOwnProperty.call(obj, prop); }
  }();

  !function () {
    __webpack_require__.r = function (exports) {
      if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
        Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
      }
      Object.defineProperty(exports, '__esModule', { value: true });
    };
  }();

  var __webpack_exports__ = __webpack_require__("./src/index.js");
})();
```

编译分析：

- __webpack_require__.r 方法，判断引入的模块是CJS还是ESM，如果是ESM会在exports上挂在一个__esModule属性，值为true，代表这是esModule。CJS不变。
- 导出：默认导出挂在exports.default上，其他属性挂在都exports上。参照__webpack_modules__下的"./src/mixModules.js” eval方法。webpack_exports[\"default\"] = (666);
- 默认导入：webpack_modules下 "./src/index.js” 获取默认导入取值使用的default属性。mixModules__WEBPACK_IMPORTED_MODULE_0_[\"default\"]

## CJS转ESM

场景：写 ESM 项目，引入了一个只有 CJS 的库时，且编译出 ESM 时，才会用到 CJS 转 ESM。

要运行 ESM 引入 CJS 的代码，有两种方式：

- 把 ESM 转 CJS，然后运行 CJS
- 把 CJS 转成 ESM，然后运行 ESM

webpack属于第一种方式，所以webpack 写 ESM，然后引入 CJS 的时候，不会遇到问题。

第二种方式，之后整理。

# 参考

[ESM和CJS模块杂谈](https://juejin.cn/post/7048276970768957477)

[前端科普系列-CommonJS：不是前端却革命了前端](https://zhuanlan.zhihu.com/p/113009496)

[Sea.js是如何工作的？](https://doc.yonyoucloud.com/doc/wiki/project/hello-seajs/how-seajs-works.html)

[终于搞懂了 ESM 和 CJS 互相转换](https://juejin.cn/post/7205897684624474168#heading-0)