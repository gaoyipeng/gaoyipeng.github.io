---
title: Vue (06) 组件化开发
date: 2020-12-29 14:08:06
tags: Vue
categories: Vue
description: Vue (06) 组件化开发
typora-root-url: ..
---

## 1、什么是组件化开发

组件化是Vue.js中的重要思想它提供了一种抽象，让我们可以开发出一个个独立可复用的小组件来构造我们的应用。任何的应用都会被抽象成一颗组件树。

![](/images/vue-06/image-20201229171742279.png)

组件化的好处：

- 代码复用及功能复用，避免重复造轮子。降低程序员的工作量.
- 方便代码的组织及管理
- 降低代码之间的耦合度，提高可扩展性
- vue组件不仅在单功能模块可以多次复用，甚至可以跨模块、跨项目复用.

## 2、全局组件和局部组件

组件分为全局组件和局部组件，组件的使用分成三个步骤：

- 创建组件构造器
- 注册组件
- 使用组件

![](/images/vue-06/image-20201230092457377.png)

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
    // 2.注册组件(局部组件，只能在当前Vue实例下使用)
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

![](/images/vue-06/image-20201230120131487.png)

- Vue.extend()

  > ​		调用Vue.extend()创建的是一个**组件构造器**。 通常在创建组件构造器时，传入template代表我们自定义组件的模板。该模板就是在使用到组件的地方，要显示的HTML代码。事实上，这种写法在Vue2.x的文档中几乎已经看不到了，大家一般使用语法糖，但是在很多资料还是会提到这种方式，而且这种方式是学习后面方式的基础。

  

- Vue.component()

  > ​       调用Vue.component()是将刚才的组件构造器注册为一个**全局组件**，并且给它起一个组件的标签名称。两个参数：注册组件的标签名 、组件构造器。

  

- components: { cpnChild：cpnChild: cpnC }

  > ​       在Vue实例下的components中添加的组件是局部组件，只能在当前实例中使用。

  

- 组件必须挂载在某个Vue实例下，否则它不会生效。

  > 上面的栗子中，我们注册了一个全局组件`cpn`，它可以在`app`和`app2`中使用，而局部组件`cpnChild`，则只能在`app`中使用。
  
  

### 2.1 组件的语法糖写法

```html
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


-----------------------------------------以上代码等价于------------------------------------------------
// 语法糖写法
Vue.component('cpn', {
    template: `
    <div>
        <h2>我是组件</h2>
    </div>
    `
})
```

### 2.2  组件模板的分离写法

我们通过语法糖简化了Vue组件的注册过程，另外还有一个地方的写法比较麻烦，就是template模块写法。如果我们能将其中的HTML分离出来写，然后挂载到对应的组件上，必然结构会变得非常清晰。Vue提供了两种方案来定义HTML模块内容：

- 使用`<script>`标签
- 使用`<template>`标签

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>

<div id="app">
  <cpn1></cpn1>
  <cpn2></cpn2>
</div>

<!--1.script标签, 注意:类型必须是text/x-template-->
<script type="text/x-template" id="cpn1">
<div>
  <h2>我是标题1</h2>
  <p>我是内容,哈哈哈</p>
</div>
</script>

<!--2.template标签-->
<template id="cpn2">
  <div>
    <h2>我是标题2</h2>
    <p>我是内容,呵呵呵</p>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>

  // 1.注册一个全局组件
  Vue.component('cpn1', {
    template: '#cpn1'
  })
  Vue.component('cpn2', {
    template: '#cpn2'
  })

  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    }
  })
</script>

</body>
</html>
```

效果如下：

![](/images/vue-06/image-20210107111934944.png)



## 3、父组件和子组件

在前面我们看到了组件树，组件和组件之间存在层级关系，而其中一种非常重要的关系就是父子组件的关系。

我们先来看一个父子组件的栗子：

```html
<body>

<div id="app">
  <cpn2></cpn2>
  <cpn1></cpn1>
</div>

<script src="../js/vue.js"></script>
<script>
  // 1.创建第一个组件构造器(子组件)
  const cpnC1 = Vue.extend({
    template: `
      <div>
        <h2>我是标题1</h2>
        <p>我是内容, 哈哈哈哈</p>
      </div>
    `
  })


  // 2.创建第二个组件构造器(父组件)
  const cpnC2 = Vue.extend({
    template: `
      <div>
        <h2>我是标题2</h2>
        <p>我是内容, 呵呵呵呵</p>
        <cpn1></cpn1>
      </div>
    `,
    components: {
      cpn1: cpnC1
    }
  })

  // root组件
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    },
    components: {
      cpn2: cpnC2
    }
  })
</script>

</body>
```

![](/images/vue-06/image-20210106175537905.png)

其中cpnC2是父组件，cpnC1是子组件。cpnC1在cpnC2的components中做了定义。

**因为当子组件注册到父组件的components时，Vue会编译好父组件的模块。父组件中已经有了子组件中的内容了。**

### 3.1 组件通信

在开发中，往往一些数据确实需要从上层传递到下层：

- 比如在一个页面中，我们从服务器请求到了很多的数据。
- 其中一部分数据，并非是我们整个页面的大组件来展示的，而是需要下面的子组件进行展示。
- 这个时候，并不会让子组件再次发送一个网络请求，而是直接让大组件(父组件)将数据传递给小组件(子组件)。

如何进行父子组件间的通信呢？Vue官方提供了如下方式：

- 通过props向子组件传递数据
- 通过事件向父组件发送消息

![](/images/vue-06/image-20210107113651373.png)

#### 3.1.1 通过props向子组件传递数据

##### 3.1.1.1 props基本的使用方法

```html

<div id="app">
  <!--<cpn v-bind:cmovies="movies"></cpn>-->
  <!--<cpn cmovies="movies" cmessage="message"></cpn>-->

  <cpn :cmessage="message" :cmovies="movies"></cpn>
</div>

<template id="cpn">
  <div>
    <ul>
      <li v-for="item in cmovies">{{item}}</li>
    </ul>
    <h2>{{cmessage}}</h2>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>
  // 父传子: props
  const cpn = {
    template: '#cpn',
     props: ['cmovies', 'cmessage']
  }

  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      movies: ['海贼王', '湄公河行动', '阿凡达']
    },
    components: {
      cpn
    }
  })
</script>
```

效果如下：

![](/images/vue-06/image-20210107142744981.png)

**注意**：在Vue2.0中，props数据流只能从父组件向子组件传递。并且在组件内，不能修改由外层传来的props数据。



##### 3.1.1.2  props数据校验

通常我们定义一个组件是应该可以提供给其他模块或其他人使用的。使用者可能对该组件的用法并不熟悉，可能会导致错误。所以有必要在子组件内，对父组件传递过来数据进行校验。验证都支持哪些数据类型呢？

- String

- Number

- Boolean

- Array

- Object

- Date

- Function

- Symbol

  下面是一些示例：

```js
Vue.component('example', {
  props: {
    // 基础类型检测 (`null` 意思是任何类型都可以)
    propA: Number,
    // 可以是多种类型
    propB: [String, Number],
    // 必传属性且必须是字符串类型
    propC: {
      type: String,
      required: true
    },
    // 数字，不传就默认值100
    propD: {
      type: Number,
      default: 100
    },
    // 数组/对象的默认值应当由一个工厂函数返回
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

当我们有自定义构造函数时，验证也支持自定义的类型:

![image-20210107154428303](/images/vue-06/image-20210107154428303.png)

##### 3.1.1.3 props中的驼峰标识的使用

我们先来看一个栗子：

```html
<body>

<div id="app">
  <cpn :c-info="info" :child-my-message="message" ></cpn>
</div>

<template id="cpn">
  <div>
    <h2>{{cInfo}}</h2>
    <h2>{{childMyMessage}}</h2>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>
  const cpn = {
    template: '#cpn',
    props: {
      cInfo: {
        type: Object,
        default() {
          return {}
        }
      },
      childMyMessage: {
        type: String,
        default: ''
      }
    }
  }

  const app = new Vue({
    el: '#app',
    data: {
      info: {
        name: 'garnett',
        age: 18,
        height: 1.88
      },
      message: 'aaaaaa'
    },
    components: {
      cpn
    }
  })
</script>

</body>
```

效果如下：

![image-20210107152721636](/images/vue-06/image-20210107152721636.png)

我们这里以`cInfo`为例说明，可以看到`<cpn :c-info="info" :child-my-message="message" ></cpn>`，中使用的是`c-info`，而template中使用的是`{ { cInfo } }`，如果我们把`c-info`改为`cInfo`,则会发现子组件无法正常获取父组件的值。此处规则如下：

- 在html代码里面标签属性命名时，使用中划线属性写法。
- 在js代码里面的变量或属性使用驼峰的写法。

#### 3.1.2 通过`$emit()`向父组件传递数据

props用于父组件向子组件传递数据，还有一种比较常见的是子组件传递数据或事件到父组件中。我们应该如何处理呢？这个时候，我们需要使用自定义事件来完成。自定义事件的流程：

- 在子组件中，通过$emit()来触发事件。
- 在父组件中，通过v-on来监听子组件事件。

```html
<body>
<!--父组件模板-->
<div id="app">
  <cpn @item-click="cpnClick"></cpn>
</div>
<!--子组件模板-->
<template id="cpn">
  <div>
    <button v-for="item in categories"
            @click="btnClick(item)">
      {{item.name}}
    </button>
  </div>
</template>
<script src="../js/vue.js"></script>
<script>

  // 1.子组件
  const cpn = {
    template: '#cpn',
    data() {
      return {
        categories: [
          {id: 'aaa', name: '热门推荐'},
          {id: 'bbb', name: '手机数码'},
          {id: 'ccc', name: '家用家电'},
          {id: 'ddd', name: '电脑办公'},
        ]
      }
    },
    methods: {
      btnClick(item) {
        // 发射事件: 自定义事件
        this.$emit('item-click', item)
      }
    }
  }

  // 2.父组件
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    },
    components: {
      cpn
    },
    methods: {
      cpnClick(item) {
        console.log('cpnClick', item.name);
      }
    }
  })
</script>

</body>
```

![](/images/vue-06/image-20210107165016002.png)

### 3.2  组件访问

前面介绍了组件通信，主要解决组件之间的数据传输。有时候我们需要父组件直接访问子组件，子组件直接访问父组件，或者是子组件访问跟组件。

- 父组件访问子组件：使用$children或$refs
- 子组件访问父组件：使用$parent

#### 3.2.1 使用$children访问子组件

```html
<body>

<div id="app">
  <cpn></cpn>
  <cpn></cpn>
  <button @click="btnClick">按钮</button>
</div>

<template id="cpn">
  <div>我是子组件</div>
</template>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    },
    methods: {
      btnClick() {
        // 1.$children
        console.log(this.$children);
        for (let c of this.$children) {
          console.log(c.name);
          c.showMessage();
        }
      }
    },
    components: {
      cpn: {
        template: '#cpn',
        data() {
          return {
            name: '我是子组件的name'
          }
        },
        methods: {
          showMessage() {
            console.log('showMessage');
          }
        }
      },
    }
  })
</script>

</body>
```

![image-20210107175821778](/images/vue-06/image-20210107175821778.png)

这样父组件就可以通过`$children`直接访问子组件，获取子组件的实例，进而操作子组件。

#### 3.2.2 使用$refs访问子组件

然而`$children`是有缺陷的：

- 通过$children访问子组件时，是一个数组类型，访问其中的子组件必须通过索引值。
- 但是当子组件过多，我们需要拿到其中一个时，往往不能确定它的索引值，甚至还可能会发生变化。

有时候，我们想明确获取其中一个特定的组件，这个时候就可以使用`$refs`。

$refs的使用：$refs和ref指令通常是一起使用的。

- 首先，我们通过ref给某一个子组件绑定一个特定的ID。
- 其次，通过this.$refs.ID就可以访问到该组件了。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>

<div id="app">
  <cpn ref="aaa"></cpn>
  <button @click="btnClick">按钮</button>
</div>

<template id="cpn">
  <div>我是子组件</div>
</template>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    },
    methods: {
      btnClick() {
        // 2.$refs => 对象类型, 默认是一个空的对象 ref='bbb'
        console.log(this.$refs.aaa.name);
        this.$refs.aaa.showMessage();
      }
    },
    components: {
      cpn: {
        template: '#cpn',
        data() {
          return {
            name: '我是子组件的name'
          }
        },
        methods: {
          showMessage() {
            console.log('showMessage');
          }
        }
      },
    }
  })
</script>

</body>
</html>
```

![image-20210107182844916](/images/vue-06/image-20210107182844916.png)

#### 3.2.3 使用$parent访问父组件

如果我们想在子组件中直接访问父组件，可以通过`$parent`

注意事项：

- 尽管在Vue开发中，我们允许通过$parent来访问父组件，但是在真实开发中尽量不要这样做。
- 子组件应该尽量避免直接访问父组件的数据，因为这样耦合度太高了。
- 如果我们将子组件放在另外一个组件之内，很可能该父组件没有对应的属性，往往会引起问题。
- 另外，更不好做的是通过$parent直接修改父组件的状态，那么父组件中的状态将变得飘忽不定，很不利于我的调试和维护。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>

<div id="app">
  <cpn></cpn>
</div>

<template id="cpn">
  <div>
    <h2>我是cpn组件</h2>
    <ccpn></ccpn>
  </div>
</template>

<template id="ccpn">
  <div>
    <h2>我是子组件</h2>
    <button @click="btnClick">按钮</button>
  </div>
</template>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊'
    },
    components: {
      cpn: {
        template: '#cpn',
        data() {
          return {
            name: '我是cpn组件的name'
          }
        },
        components: {
          ccpn: {
            template: '#ccpn',
            methods: {
              btnClick() {
                // 1.访问父组件$parent
                console.log(this.$parent);
                console.log(this.$parent.name);

                // 2.访问根组件$root
                console.log(this.$root);
                console.log(this.$root.message);
              }
            }
          }
        }
      }
    }
  })
</script>

</body>
</html>
```

![image-20210107183603728](/images/vue-06/image-20210107183603728.png)

## 4、插槽