---
layout: post
title: 前端JavaScript模块发展心酸历程
date: 2019-09-27 14:32:20 +0800
description: 前端JavaScript模块的发展心酸历程 # Add post description (optional)
img: i-rest.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [JavaScript modules, frontend]
---
历史发展：
JavaScript这门语言从问世到现在，已经有20年历史了，在这20年中，语言本身语法也在不断演变，不断更新，从简单的应用场景到现在spa单页应用。

前端开发者在这个历程中经历了太多的曲折，今天从模块化角度简单探讨下JavaScript模块化（注意：不是前端模块化）的发展历程。

JavaScript定位：
早起的JavaScript语言问世时，定位是辅助html在浏览器上的展示并实现简单的表单验证提交、简单的动画展示，因此JavaScript并没有被设计的很复杂，并没有模块化的思考。

# 第一阶段：无模块化

在1999年，绝大部分工程师做JavaScript开发时就直接将变量定义在全局，这种方式成为*直接定义依赖*

代码类似下面这样
```
// other1.js
var helloInLang = {
  en: 'Hello world!',
  es: '¡Hola mundo!'
};
function writeHello(lang) {
  document.write(helloInLang[lang]);
}

// other2.js
function writeHello() {
  document.write('The script is broken');
}

// index.html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <script src="other1.js"></script>
  <script src="other2.js"></script>
</head>
<body onLoad="writeHello('en')">
</body>
</html>
```
缺点：
* 每个模块都直接暴露在全局中，供直接使用，容易造成变量冲突及全局作用域污染

# 第二阶段：命名空间

在2002年，有人提出了以命名空间为思路，用于解决遍地的全局变量，将需要定义的部分归属到一个对象的属性上，这种模式称为*命名空间模式*
代码类似下面这样

```
// other1.js
var app = {};
app.helloInLang = {
  en: 'Hello world!',
  es: '¡Hola mundo!'
};
app.writeHello = function (lang) {
  document.write(helloInLang[lang]);
}

// other2.js
function writeHello() {
  document.write('The script is broken');
}
```
缺点：
* 本质上并没有解决全局作用域污染，只是通过减少全局变量的定义，尽可能减少代码冲突
* 全局对象，任何人都可以访问并操作，容易造成误操作

# 第三阶段：闭包模块化
在2003年，提出利用立即调用函数（IIFE）和闭包（Closures）特性，解决私有变量问题，这种模式称为*闭包模块化模块*

## 3.1 手动实现
```
// other1.js
var greeting = (function() {
  var module = {};
  var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!'
  };

  module.getHello = function(lang) {
    return helloInLang[lang];
  };

  module.writeHello = function(lang) {
    document.write(module.getHello(lang));
  };

  return module;
})();
```

缺点：
* 缺少一个管理者，管理各模块之间的依赖关系，在模块较多的时候，需要手动维护加载的先后顺序，容易出现依赖错误。

## 3.2 同步依赖实现

### 3.2.1 CommonJS规范实现
在2009年，真正的革命来了，CommonJS规范的落地，目的是让浏览器之外的JavaScript能够通过模块化的方式来开发和协作。此时前端开始大量使用预编译。

在CommonJS规范中，每个JavaScript文件就是一个独立的模块上下文，在这个上下文中默认创建的属性都是私有的，其他文件不可见，如果想将当前文件的属性暴露出去，通过module.exports 对象来暴露对外的接口，Node 就是采用CommonJS规范实现模块依赖。

```
// a.js
function a() {
  return {
    hello: function() {
      ...
    },
    goodbye: function() {
      ...
    }
  };
}
module.exports = a;
```

```
// b.js
var a = require('a');
a.hello();
a.goodbye();
```
优点：
* 避免了全局命名空间污染
* 有清晰的模块依赖关系

缺点：
* 同步加载依赖项，在服务端由于模块来源是硬盘、内存等，加载速度快，同步加载无问题。浏览器场景下，由于需要发起网络请求，需要网络耗时及等待，因此不适用浏览器端。

## 3.3 异步依赖实现
### 3.3.1 AMD规范实现
AMD是Asynchronous Module Definition的简称，即“异步模块定义”，其中RequireJS就是基于AMD的实现。

顾名思义，在模块依赖加载时，会以一种非阻塞的方式，通过appendChild将多个依赖的模块插入到DOM中，在依赖模块加载成功之后，调用回调函数实现异步的等待执行。

```
// 有依赖的模块定义
define(['moduleA','moduleB'], function(moduleA,moduleB) {
  return {
    a: function() {
      ...
    },
    b: function() {
      ...
    }
  }
})

// 无依赖的模块定义
define([], function() {
  return {
    a: function() {
      ...
    },
    b: function() {
      ...
    }
  }
})
```
特点：
* 依赖前置
* 提前执行
所以我们看到，AMD模式，优先照顾浏览器的模块加载场景，使用了异步的加载和回调方式，跟CommonJS截然不同。

### 3.3.2 CMD规范实现
CMD是Common Module Definition的简称，即“通用模块定义”，其中SeaJS就是基于CMD的实现。

```
define(function(require,exports,module){
  var other1=require('./other1')
  other1.doSomethimg()
  var other2=require('./other2')
  other2.doSomething()
})
```
特点：
* 就近依赖，当模块有依赖模块时，不需要前置引入，仅在使用到的地方引入依赖模块即可。
* 延迟执行，所依赖的模块已提前加载进来，但是不会执行模块内部的代码，等到使用到该模块时，才执行模块内部代码。

小结：
* AMD、CMD都实现了define、require及module的核心功能，虽然实现的思路有些区别，AMD偏向依赖前置，CMD偏向就近依赖。但都是异步依赖的实现，适用于浏览器环境。

### 3.4 UMD规范实现
UMD是Universal Module Definition，实现CommonJS和AMD的统一

原理如下：
在执行UMD规范时，优先判断当前环境是否支持AMD，然后判断是否支持CommonJS，最后认为当前环境为浏览器window环境
```
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD
    define(['myModule', 'myOtherModule'], factory);
  } else if (typeof exports === 'object') {
    // CommonJS
    module.exports = factory(require('myModule'), require('myOtherModule'));
  } else {
    // Browser globals
    root.returnExports = factory(root.myModule, root.myOtherModule);
  }
}(this, function (myModule, myOtherModule) {
  function hello(){}; 
  function goodbye(){};

  return {
      hello: hello,
      goodbye: goodbye
  }
}));
```

# 第四阶段：编译时加载-ES6原生实现
可能大家已经注意到了，上面提到的所有方法，都不是JavaScript语言原生支持的。但是ES6标准出来之后，已经引入了原生的模块功能。

ES6的原生模块功能非常棒，它兼顾了规范、语法简约性和异步加载功能，并且还支持循环依赖，实现编译时确定模块的依赖关系。

最厉害的是，import进来的模块对于调用它的模块来说，是一个活的只读视图，而不是想CommonJS一样是一个内存的拷贝。
下面是ES6模块的实例：

```
// lib/a.js
export let a = 1;
export function add() {
  a++;
};

// main.js
import * as a from 'lib/a.js';
console.log(a.a);
a.add();
console.log(a.a);
```

# 总结
ES6的模块语法，虽然在主流浏览器还没支持，但是可以通过Babel等转译器实现转译，因此可以放心使用。