---
title: Vue父子组件通信
date: 2020-12-29 14:08:06
tags: Vue
categories: Vue
description: Vue父子组件通信
typora-root-url: ..
---

## 1、什么是组件化开发

组件化是Vue.js中的重要思想它提供了一种抽象，让我们可以开发出一个个独立可复用的小组件来构造我们的应用。任何的应用都会被抽象成一颗组件树。

![image-20201229171742279](/images/vue-06/image-20201229171742279.png)

组件化的好处：

- 代码复用及功能复用，避免重复造轮子。降低程序员的工作量.
- 方便代码的组织及管理
- 降低代码之间的耦合度，提高可扩展性
- vue组件不仅在单功能模块可以多次复用，甚至可以跨模块、跨项目复用.

## 2、全局组件和局部组件

组件的使用分成三个步骤：

- 创建组件构造器
- 注册组件
- 使用组件

![image-20201230092457377](/images/vue-06/image-20201230092457377.png)

```html
<body>

<div id="app">
  全局组件：<cpn></cpn>
  局部组件：<cpn-child></cpn-child>
</div>

<br/><br/><br/>

<div id="app2">
  全局组件：<cpn></cpn>
  局部组件: <cpn-child></cpn-child>
</div>

<script src="../js/vue.js"></script>
<script>
  // 1.创建组件构造器
  const cpnC = Vue.extend({
    template: `
      <div>
        <h2>我是组件</h2>
      </div>
    `
  })

  // 2.注册组件(全局组件, 意味着可以在多个Vue的实例下面使用)
   Vue.component('cpn', cpnC)

  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    },
    // 2.注册组件(局部组件, 意味着可以在多个Vue的实例下面使用)
    components: {
      // cpnChild：使用组件时的标签名
      cpnChild: cpnC
    }
  })

  const app2 = new Vue({
    el: '#app2'
  })
</script>

</body>
```

![image-20201230120131487](/images/vue-06/image-20201230120131487.png)

- Vue.extend()

  调用Vue.extend()创建的是一个**组件构造器**。 通常在创建组件构造器时，传入template代表我们自定义组件的模板。该模板就是在使用到组件的地方，要显示的HTML代码。事实上，这种写法在Vue2.x的文档中几乎已经看不到了，大家一般使用语法糖，但是在很多资料还是会提到这种方式，而且这种方式是学习后面方式的基础。

- Vue.component()

  调用Vue.component()是将刚才的组件构造器注册为一个**全局组件**，并且给它起一个组件的标签名称。
  两个参数：注册组件的标签名 、组件构造器

- components: { cpnChild：cpnChild: cpnC }

  在Vue实例下的components中添加的组件是局部组件，只能在当前实例中使用。

- 组件必须挂载在某个Vue实例下，否则它不会生效。上面的栗子中，我们注册了一个全局组件`cpn`，它可以在`app`和`app2`中使用，而局部组件`cpnChild`，则只能在`app`中使用。



