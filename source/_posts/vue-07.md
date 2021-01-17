---
title: Vue (07) 模块化开发
date: 2021-01-14 10:10:34
tags: Vue
categories: Vue
description: Vue (07) 模块化开发
typora-root-url: ..
---

## 1、模块化解决方案

不解释模块化的出现的意义，如果是用过`Query`的童鞋一定能想起被几十个`.js`文件支配的恐惧。

- AMD 提供了异步加载的功能的JavaScript模块化规范，RequireJS是实现了该规范的类库
- CMD 最初是由阿里的玉伯提出的，同AMD类似，使用CMD模块也需要使用对应的SeaJS类库
- **CommonJS** 这是一种被广泛使用的Javascript模块化规范，大家最熟悉的Node.js应用中就是采用这个规范
- UMD 通用模块定义(Universal Module Definition)，使用该模块化方案，可以很好地兼容AMD， CommonJS等模块化语法。
- **ES6 Modules**，ES6全称 ECMAScript 6.0 ,是 JavaScript 的版本标准。也就是javascript官方的亲儿子，也就是说未来会受到众多浏览器的支撑！即使目前浏览器还无法完全支持ES6 modules，我们还是可以通过Babel等转译工具进行编译使用ES6 Modules。

我们介绍到的打包工具webpack是依赖于NodeJS的，所以我们很有必要学习一下CommonJS模块化管理。另外，ES6 Modules模块化解决方案时一定要学的。随着ES6 Modules的推出，AMD、CMD、UMD会逐渐退出历史舞台。

## 2、**CommonJS** 

模块化有两个核心。

- 导出示例

![image-20210116165837892](/images/vue-07/image-20210116165837892.png)

- 导入示例

![image-20210116165859942](/images/vue-07/image-20210116165859942.png)

CommonJS的模块管理方案是依赖于NodeJS环境的，所以上面的代码是无法在浏览器端执行的。那我们vue课程为什么还要学？Vue用到的打包工具webpack，是依赖于NodeJS使用CommonJS做模块化管理方案的。

## 3、ES6 Modules

### 3.1 ES6 Modules导出方式



第一种方式，先定义变量及函数、类，然后使用export {……} 进行统一导出。

第二种方式，在定义变量及函数、类的时候就使用`export` 关键字导出。

我们新建一个module-a.js：

![image-20210116180028347](/images/vue-07/image-20210116180028347.png)



### 3.2  ES6 Modules导入方式 

我们使用export指令导出了模块对外提供的接口，下面我们就可以通过import命令来加载对应的这个模块了。

在module-c.js中使用import {……}语法导入变量、函数、类，from关键字后面跟上导入模块文件的路径。

![image-20210116185727132](/images/vue-07/image-20210116185727132.png)

如果导出的内容比较多，我们可以使用 import * as 的语法为到处内容起一个别名，如下图中的moduleA。然后在导入之后的使用过程都要用："moduleA."的语法引用变量、函数、类。

![image-20210116185815590](/images/vue-07/image-20210116185815590.png)

某些情况下，一个模块中包含某个的功能，我们并不希望给这个功能命名，而且让导入者可以自己来命名
这个时候就可以使用**export default**方式导出

![image-20210116185244923](/images/vue-07/image-20210116185244923.png)

我们来到main.js中，这样使用就可以了这里的myFunc是自己命名的，可以根据需要命名它对应的名字

![image-20210117161955408](/images/vue-07/image-20210117161955408.png)

另外，需要注意：export default在同一个模块中，不允许同时存在多个。

### 3.3  html 导入模块

3.1和3.2都是在Vue环境中导入的，如果我们想要在html文件中引用模块化js文件应该怎么办呢？

```js
<script src="module-c.js" type="module"></script>
```



