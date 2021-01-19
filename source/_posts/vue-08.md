---
title: Vue (08) webpack学习
date: 2021-01-18 13:59:57
tags: Vue
categories: Vue
description: Vue (08) webpack学习
typora-root-url: ..
---

## 1、为什么使用webpack

上一节我们引入了模块化的概念，学习了CommonJS 和 ES6 Modules。但是有一些问题：

1. 如CommonJS这种模块化方案代码，浏览器是无法直接执行的。虽然ES6的模块化导入导出代码可以被大部分浏览器所支持，但是仍有少部分的浏览器无法支持
2. 模块之间有依赖关系，如何自动将有依赖关系的模块打包。
3. 兼容前端的新技术：LESS、SASS、Stylus、PostCSS、vue、react、Angular、CoffeScript和TypeScript。这些前端技术是无法直接被浏览器识别的，例如`.vue``.less`文件等，浏览器是无法直接展示的。

那怎么办呢？webpack可以帮我们把CommonJS、ES6以及上面列举的各种技术（vue、react等）转换为浏览器可以解释执行的语法。

也就是说，我们既可以使用到模块化方案带飞我们的好处：避免代码变量命名等冲突，结构清晰，方便复用与管理，也可以通过webpack来满足浏览器的兼容！webpack可以帮我们解析模块间的依赖关系，并正确打包！

除此之外，webpack还有以下功能：

- 压缩代码、丑化代码、难以被识别
- webpack帮我们管理小素材图标，以base64字符串格式存储等



## 2、如何使用webpack

webpack是依赖nodeJS的。

**nodeJS、webpack、npm、vue是什么关系？**

- vue，实际上是学习js语言一种新的框架。这个新的框架里面有一部分语法是不被浏览器兼容的，比如“.vue”单文件，我们就需要webpack来帮我们做一个翻译，翻译后的代码浏览器可执行。
- 我们在开发前端项目的过程中，可能会用到各种js类库，npm工具可以帮助我们来管理这些包，打包工具webpack本身就是一个js的类库，需要使用npm帮我们管理。可以把 npm看成 maven中央仓库，用来存储管理各种js类库。
- webpack是一个js类库，除了帮我们做翻译，还帮我们打包静态资源。我们都知道js代码是需要执行环境的，浏览器环境或者NodeJS环境。所以webpack依赖于NodeJS环境。