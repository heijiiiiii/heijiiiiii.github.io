---
layout: post
title: 前端js模块发展心酸历程
date: 2019-09-27 14:32:20 +0800
description: 前端js模块的发展心酸历程 # Add post description (optional)
img: i-rest.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [js modules, frontend]
---
历史发展：
js这门语言从问世到现在，已经有20年历史了，在这20年中，语言本身语法也在不断演变，不断更新，从简单的应用场景到现在spa单页应用。

前端开发者在这个历程中经历了太多的曲折，今天从模块化角度简单探讨下js模块化（注意：不是前端模块化）的发展历程。

js定位：
早起的js语言问世时，定位是辅助html在浏览器上的展示并实现简单的表单验证提交、简单的动画展示，因此js并没有被设计的很复杂，并没有模块化的思考。

# 第一阶段：无模块化
js发展面临的问题：
随着AJAX技术的问世，前端开发者已经慢慢感受到前端逻辑的复杂性，js的代码量也在逐日增多。在项目较小的场景下，甚至只使用一个js文件将所有功能逻辑放在一起。但是随着项目体量的越来越大以及开源库的发展，开发者需要引入外部开源库时（如jQuery等库），就容易带来文件的依赖以及冲突。

解决方案：
简单的将所有需要用到的js文件统一引入到html文件中

代码类似下面这样
```
<script scr="jqeury.js"></script>
<script src="jquery.scroller.js"></script>
<script src="main.js"></script>
<script src="other1.js></script>
<script src="other2.js></script>
```
缺点：
* 每个模块都直接暴露在全局中，供直接使用，容易造成变量冲突及全局作用域污染
* 当项目的js特别多的时候，需要手动维护js的加载先后循序，避免出现依赖出错
* 依赖关系不直观，不利于维护

思考：
开发者迫切的需要一种方案能解决全局作用域污染、维护js依赖关系。这个时候，用函数的形式实现一个模块化加载就出现了。

# 第二阶段：函数实现
特点：用函数的闭包，实现模块的同步异步加载，维护依赖关系

## 个人封装
### 使用匿名函数
```
(function() {
  var a = [];
  var b = function() {
    ...
  }
})();
```
### 将全局函数注入到匿名函数
```
(function(globalVariable) {
  globalVariable.a = function() {
    ...
  }
})(globalVariable)
```
### 提供一个对象作为接口
```
var a = (function() {
  var b = [];
  return {
    c: function() {
      ...
    },
    d: function() {
      ...
    }
  };
})()
```
以上的方式，是利用函数的闭包特性，目的都是减少变量对全局的污染，但并没有解决模块依赖的头疼问题。

## CommonJS规范实现
CommonJS目的是让浏览器之外的JavaScript能够通过模块化的方式来开发和协作。

在CommonJS规范中，每个JavaScript文件就是一个独立的模块上下文，在这个上下文中默认创建的属性都是私有的，其他文件不可见，如果想将当前文件的属性暴露出去，通过module.exports 对象来暴露对外的接口，Node 就是采用CommonJS规范实现模块依赖。

`a.js`
```
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

`b.js`
```
var a = require('a');
a.hello();
a.goodbye();
```
优点：
* 避免了全局命名空间污染
* 有清晰的模块依赖关系

缺点：
* 同步加载依赖项，在服务端由于模块来源是硬盘、内存等，加载速度快，同步加载无问题。浏览器场景下，由于需要发起网络请求，需要网络耗时及等待，因此不适用浏览器端。

## AMD规范实现
AMD是Asynchronous Module Definition的简称，即“异步模块定义”。

顾名思义，在模块依赖加载时，会以一种非阻塞的方式，通过appendChild将多个依赖的模块插入到DOM中，在依赖模块加载成功之后，调用回调函数实现异步的等待执行。

`有依赖的模块定义`
```
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
```

`无依赖的模块定义`
```
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
所以我们看到，AMD模式，优先照顾浏览器的模块加载场景，使用了异步的加载和回调方式，跟CommonJS截然不同。
## UMD规范实现
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

# 第三阶段：ES6原生实现
可能大家已经注意到了，上面提到的所有方法，都不是JavaScript语言原生支持的。但是ES6标准出来之后，已经引入了原生的模块功能。

ES6的原生模块功能非常棒，它兼顾了规范、语法简约性和异步加载功能，并且还支持循环依赖。

最厉害的是，import进来的模块对于调用它的模块来说，是一个活的只读视图，而不是想CommonJS一样是一个内存的拷贝。
下面是ES6模块的实例：

`lib/a.js`
```
export let a = 1;
export function add() {
  a++;
};
```
`main.js`
```
import * as a from 'lib/a.js';
console.log(a.a);
a.add();
console.log(a.a);
```

# 总结
ES6的模块语法，虽然在主流浏览器还没支持，但是可以通过Babel等转译器实现转译，因此可以放心使用。