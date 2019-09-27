---
layout: post
title: 前端js模块发展历程
date: 2019-09-27 14:32:20 +0800
description: 前端js模块的发展历程. # Add post description (optional)
img: i-rest.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [js modules, frontend]
---
历史发展：
js这门语言从问世到现在，已经有20年历史了，在这20年中，语言本身语法也在不断演变，不断更新，从简单的应用场景到现在spa单页应用。前端开发者在这个历程中经历了太多的曲折，今天从模块化角度简单探讨下js模块化（注意：不是前端模块化）的发展历程。

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


![I and My friends]({{site.baseurl}}/assets/img/we-in-rest.jpg)

Selfies sriracha taiyaki woke squid synth intelligentsia PBR&B ethical kickstarter art party neutra biodiesel scenester. Health goth kogi VHS fashion axe glossier disrupt, vegan quinoa. Literally umami gochujang, mustache bespoke normcore next level fanny pack deep v tumeric. Shaman vegan affogato chambray. Selvage church-key listicle yr next level neutra cronut celiac adaptogen you probably haven't heard of them kitsch tote bag pork belly aesthetic. Succulents wolf stumptown art party poutine. Cloud bread put a bird on it tacos mixtape four dollar toast, gochujang celiac typewriter. Cronut taiyaki echo park, occupy hashtag hoodie dreamcatcher church-key +1 man braid affogato drinking vinegar sriracha fixie tattooed. Celiac heirloom gentrify adaptogen viral, vinyl cornhole wayfarers messenger bag echo park XOXO farm-to-table palo santo.

>Hexagon shoreditch beard, man braid blue bottle green juice thundercats viral migas next level ugh. Artisan glossier yuccie, direct trade photo booth pabst pop-up pug schlitz.

Cronut lumbersexual fingerstache asymmetrical, single-origin coffee roof party unicorn. Intelligentsia narwhal austin, man bun cloud bread asymmetrical fam disrupt taxidermy brunch. Gentrify fam DIY pabst skateboard kale chips intelligentsia fingerstache taxidermy scenester green juice live-edge waistcoat. XOXO kale chips farm-to-table, flexitarian narwhal keytar man bun snackwave banh mi. Semiotics pickled taiyaki cliche cold-pressed. Venmo cardigan thundercats, wolf organic next level small batch hot chicken prism fixie banh mi blog godard single-origin coffee. Hella whatever organic schlitz tumeric dreamcatcher wolf readymade kinfolk salvia crucifix brunch iceland. Literally meditation four loko trust fund. Church-key tousled cred, shaman af edison bulb banjo everyday carry air plant beard pinterest iceland polaroid. Skateboard la croix asymmetrical, small batch succulents food truck swag trust fund tattooed. Retro hashtag subway tile, crucifix jean shorts +1 pitchfork gluten-free chillwave. Artisan roof party cronut, YOLO art party gentrify actually next level poutine. Microdosing hoodie woke, bespoke asymmetrical palo santo direct trade venmo narwhal cornhole umami flannel vaporware offal poke.

* Hexagon shoreditch beard
* Intelligentsia narwhal austin
* Literally meditation four
* Microdosing hoodie woke

Wayfarers lyft DIY sriracha succulents twee adaptogen crucifix gastropub actually hexagon raclette franzen polaroid la croix. Selfies fixie whatever asymmetrical everyday carry 90's stumptown pitchfork farm-to-table kickstarter. Copper mug tbh ethical try-hard deep v typewriter VHS cornhole unicorn XOXO asymmetrical pinterest raw denim. Skateboard small batch man bun polaroid neutra. Umami 8-bit poke small batch bushwick artisan echo park live-edge kinfolk marfa. Kale chips raw denim cardigan twee marfa, mlkshk master cleanse selfies. Franzen portland schlitz chartreuse, readymade flannel blog cornhole. Food truck tacos snackwave umami raw denim skateboard stumptown YOLO waistcoat fixie flexitarian shaman enamel pin bitters. Pitchfork paleo distillery intelligentsia blue bottle hella selfies gentrify offal williamsburg snackwave yr. Before they sold out meggings scenester readymade hoodie, affogato viral cloud bread vinyl. Thundercats man bun sriracha, neutra swag knausgaard jean shorts. Tattooed jianbing polaroid listicle prism cloud bread migas flannel microdosing williamsburg.

Echo park try-hard irony tbh vegan pok pok. Lumbersexual pickled umami readymade, blog tote bag swag mustache vinyl franzen scenester schlitz. Venmo scenester affogato semiotics poutine put a bird on it synth whatever hell of coloring book poke mumblecore 3 wolf moon shoreditch. Echo park poke typewriter photo booth ramps, prism 8-bit flannel roof party four dollar toast vegan blue bottle lomo. Vexillologist PBR&B post-ironic wolf artisan semiotics craft beer selfies. Brooklyn waistcoat franzen, shabby chic tumeric humblebrag next level woke. Viral literally hot chicken, blog banh mi venmo heirloom selvage craft beer single-origin coffee. Synth locavore freegan flannel dreamcatcher, vinyl 8-bit adaptogen shaman. Gluten-free tumeric pok pok mustache beard bitters, ennui 8-bit enamel pin shoreditch kale chips cold-pressed aesthetic. Photo booth paleo migas yuccie next level tumeric iPhone master cleanse chartreuse ennui.