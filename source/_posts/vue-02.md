---
title: Vue指令学习
date: 2020-12-09 10:44:07
tags: Vue
categories: Vue
description: Vue指令学习
typora-root-url: ..
---

## 一、Vue指令大全

### 1、页面插值指令

#### 1.1、v-html

- 如果我们想将html原样输出，就是用{{}}就可以了

- 如果我们希望浏览器解析html后展示，那就要使用到v-html了

  ```html
  <div id="app">
      <h2>{{baidu}}</h2>
      <h2 v-html="baidu"></h2>
    </div>
    
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
    <script>
      const app = new Vue({
        el: '#app',
        data: {
          baidu: '<a href="https://www.baidu.com/">百度</a>'
        }
      })
    </script>
  ```

#### 1.2、v-pre

- Mustache原样输出，而不是交给浏览器做变量插值操作，这就需要用到v-pre指令,例如

  ```html
  <div id="app" style="background-color:aquamarine">
      <h2 v-pre>{{Mustache}}会按字符串输出，Vue不会去解析{{}}中的内容</h2>
  </div>
  ```

#### 1.3、v-text

- v-text是用于操作纯文本，它会替代显示对应的数据对象上的值。当绑定的数据对象上的值发生改变，插值处的内容也会随之更新。注意：此处为单向绑定，数据对象上的值改变，插值会发生变化；但是当插值发生变化并不会影响数据对象的值。**v-text可以简写为{{}},并且支持逻辑运算**。

- v-text 可以解决{{}}带来的闪屏问题。不过v-text在一定程度上v-text失去了{{}}使用语法的灵活性

  ```html
  <div id="app">
     <h2>{{message}}</h2>
      等价于:
      <h2 v-text="message"></h2>
  </div>
  var app = new Vue({
     el : '#app',
     data : {
      message : 'hello world'    
   }
  })
  ```

#### 1.4、v-cloak

- v-cloak被用来解决闪屏的问题

  ![image-20201209145502055](/images/vue-02/image-20201209145502055.png)

- 使用v-cloak指令并为该指令设置`display：none`的样式，二者结合使用表示在h2的Dom节点渲染完成之前不显示`你好啊，{{message}}`,渲染完成之后显示`你好啊，VUE`

- 另外要注意：样式的定义要放在head标签里面，否则因为浏览器渲染顺序的差异，有可能不会生效。

#### 1.5、v-once

-  如果显示的信息后续不需要再修改，使用v-once，这样可以提高性能。

  ```html
  <div id="app">
    <h2>{{message}}</h2>
    <h2 v-once>{{message}}</h2>
  </div>
  
  <script src="../js/vue.js"></script>
  <script>
    const app = new Vue({
      el: '#app',
      data: {
        message: '你好啊'
      }
    })
  </script>
  
  ```

  当修改data.message时，添加了v-once的h2标签内容不会修改。

### 2、数据绑定指令

#### 2.1、v-bind

- v-bind用于绑定一个或多个属性值，或者向另一个组件传递props值

![image-20201209152633111](/images/vue-02/image-20201209152633111.png)

上面介绍了v-bind的基本使用方法，以及它的语法糖写法。v-bind绑定元素除了进行单个属性值绑定，还可以传入**对象**和**数组**。

##### 2.1.1 v-bind绑定class属性

我们可以利用v-bind:class来绑定一些css属性。

###### 2.1.1.1 对象语法

对象语法的含义是:class后面跟的是一个对象。对象语法有下面这些用法：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
  <style>
    .active {
      color: red;
    }
  </style>
</head>
<body>

<div id="app">
	用法一：直接通过{}绑定一个类
	<h2 :class="{'active': isActive}">Hello World</h2>

	用法二：也可以通过判断，传入多个值
	<h2 :class="{'active': isActive, 'line': isLine}">Hello World</h2>

	用法三：和普通的类同时存在，并不冲突
	注：如果isActive和isLine都为true，那么会有title/active/line三个类
	<h2 class="title" :class="{'active': isActive, 'line': isLine}">Hello World</h2>

	用法四：如果过于复杂，可以放在一个methods或者computed中,注：classes是一个计算属性(后面介绍)
	<h2 class="title" :class="classes">Hello World</h2>
 	<button v-on:click="btnClick">按钮</button>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      isActive: true,
      isLine: true
    },
    methods: {
      btnClick: function () {
        this.isActive = !this.isActive
      },
      getClasses: function () {
        return {active: this.isActive, line: this.isLine}
      }
    }
  })
</script>

</body>
</html>
```

###### 2.1.1.2 数组语法

数组语法的含义是:class后面跟的是一个数组。数组语法有下面这些用法：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
  <style>
    .active {
      color: red;
    }
  </style>
</head>
<body>
<div id="app">
	用法一：直接通过{}绑定一个类
 	<h2 :class="['active']">Hello World</h2>

	用法二：也可以传入多个值
	<h2 :class="['active', 'line']">Hello World</h2>

	用法三：和普通的类同时存在，并不冲突 注：会有title、active、line三个类                  
	<h2 class="title" :class="['active', 'line']">Hello World</h2>

	用法四：如果过于复杂，可以放在一个methods或者computed中 注：classes是一个计算属性(后面介绍)
	<h2 class="title" :class="classes">Hello World</h2>
    <button v-on:click="btnClick">按钮</button>
</div>
<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      isActive: true,
      isLine: true
    },
    methods: {
      btnClick: function () {
        this.isActive = !this.isActive
      },
      getClasses: function () {
        return {active: this.isActive, line: this.isLine}
      }
    }
  })
</script>

</body>
</html>
```

##### 2.1.2 v-bind绑定style属性

我们可以利用v-bind:style来绑定一些CSS内联样式。

###### 2.1.2.1 对象语法

```html
<div :style="{color: baseStyle, fontSize: fontSize + 'px'}"></div>

<script>
  const app = new Vue({
    el: '#app',
    data: {
      baseStyle: 'red',
      fontSize： 10,
    }
  })
</script>
```

说明：style后面跟的是一个对象类型。对象的key是CSS属性名称,对象的value是具体赋的值，值可以来自于data中的属性

###### 2.1.2.2 数组语法

```html
<div v-bind:style="[baseStyle, baseStyle1]"></div>

<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      baseStyle: {backgroundColor: 'red'},
      baseStyle1: {fontSize: '100px'},
    }
  })
</script>
```

说明：style后面跟的是一个数组类型，多个值以，分割即可。