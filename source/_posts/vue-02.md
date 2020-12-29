---
title: Vue指令学习
date: 2020-12-09 10:44:07
tags: Vue
categories: Vue
description: Vue指令学习
typora-root-url: ..
---

## 一、Vue常用指令大全

### 1、插值指令

#### 1.1、v-html

- 如果我们想将`html`原样输出，就是用{ { } }就可以了

- 如果我们希望浏览器解析html后展示，那就要使用到`v-html`了

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

- Mustache原样输出，而不是交给浏览器做变量插值操作，这就需要用到`v-pre`指令,例如

  ```html
  <div id="app" style="background-color:aquamarine">
      <h2 v-pre>{{Mustache}}会按字符串输出，Vue不会去解析{{}}中的内容</h2>
  </div>
  ```

#### 1.3、v-text

- v-text是用于操作纯文本，它会替代显示对应的数据对象上的值。当绑定的数据对象上的值发生改变，插值处的内容也会随之更新。注意：此处为单向绑定，数据对象上的值改变，插值会发生变化；但是当插值发生变化并不会影响数据对象的值。**v-text可以简写为{ { } },并且支持逻辑运算**。

- v-text 可以解决`{ { } }`带来的闪屏问题。不过`v-text`在一定程度上v-text失去了`{ { } }`使用语法的灵活性

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



#### 2.2 v-model

前面我们学习了`v-bind`，可以通过模型数据去影响视图。我们都知道Vue是支持双向数据绑定的，那么视图是如何影响数据的呢？Vue中使用v-model指令来实现表单元素和数据的双向绑定。v-model其实是一个语法糖，它的背后本质上是包含两个操作：

- 1.v-bind绑定一个value属性

- 2.v-on指令给当前元素绑定input事件

  ```html
  <input type="text" v-model="message">
  等同于
  <input type="text" v-bind:value="message" v-on:input="message = $event.target.value">
  ```

##### 2.2.1 v-model的基本使用

```html
<div id="app">
  <input type="text" v-model="message">
  {{message}}
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

![image-20201221151130670](/images/vue-02/image-20201221151130670.png)

##### 2.2.2 v-model绑定radio和checkbox

在表单操作中，我们不只要收集文本输入的数据，我们还可能用到单选和多选。通常，实现单选和多选有以下的方式：

- 第一种：input标签type=radio实现单选和type=checkbox实现的多选

```html
<body>

<div id="app">
  <label for="male">
    <input type="radio" id="male" value="男" v-model="sex">男
  </label>
  <label for="female">
    <input type="radio" id="female" value="女" v-model="sex">女
  </label>
  <h2>您选择的性别是: {{sex}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      sex: '女'
    }
  })
</script>

</body>
```

![image-20201221152215292](/images/vue-02/image-20201221152215292.png)

```html
<body>
<div id="app">
  1.checkbox单选框
  <label for="agree">
    <input type="checkbox" id="agree" v-model="isAgree">同意协议
  </label>
  <h2>您选择的是: {{isAgree}}</h2>
  <button :disabled="!isAgree">下一步</button>

  <br/><p> </p>

  <!--2.checkbox多选框-->
  <input type="checkbox" value="篮球" v-model="hobbies">篮球
  <input type="checkbox" value="足球" v-model="hobbies">足球
  <input type="checkbox" value="乒乓球" v-model="hobbies">乒乓球
  <input type="checkbox" value="羽毛球" v-model="hobbies">羽毛球
  <h2>您的爱好是: {{hobbies}}</h2>

  <label v-for="item in originHobbies" :for="item">
    <input type="checkbox" :value="item" :id="item" v-model="hobbies">{{item}}
  </label>
</div>
<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      isAgree: false, // 单选框
      hobbies: [], // 多选框,
      originHobbies: ['篮球', '足球', '乒乓球', '羽毛球', '台球', '高尔夫球']
    }
  })
</script>

</body>
```

![image-20201221152732894](/images/vue-02/image-20201221152732894.png)

1. 使用radio实现单选，v-model绑定的数据应该是同一个，这样实现单选选项之间的互斥
2. 使用checkbox实现多选，v-model绑定的数据应该是同一个，这样多选的数据才属于同一个数组

- 第二种：select标签-option标签实现的单选与多选

```html
<body>

<div id="app">
  <!--1.选择一个-->
  <select name="abc" v-model="fruit">
    <option value="苹果">苹果</option>
    <option value="香蕉">香蕉</option>
    <option value="榴莲">榴莲</option>
    <option value="葡萄">葡萄</option>
  </select>
  <h2>您选择的水果是: {{fruit}}</h2>

  <!--2.选择多个-->
  <select name="abc" v-model="fruits" multiple>
    <option value="苹果">苹果</option>
    <option value="香蕉">香蕉</option>
    <option value="榴莲">榴莲</option>
    <option value="葡萄">葡萄</option>
  </select>
  <h2>您选择的水果是: {{fruits}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      fruit: '香蕉',
      fruits: []
    }
  })
</script>

</body>
```

![image-20201221154430087](/images/vue-02/image-20201221154430087.png)

##### 2.2.3 v-model的修饰符

- lazy修饰符：

  默认情况下，v-model默认是在input事件中同步输入框的数据的。也就是说，一旦有数据发生改变对应的data中的数据就会自动发生改变。lazy修饰符可以让数据在失去焦点或者回车时才会更新。

- number修饰符：
  默认情况下，在输入框中无论我们输入的是字母还是数字，都会被当做字符串类型进行处理。但是如果我们希望处理的是数字类型，那么最好直接将内容当做数字处理。number修饰符可以让在输入框中输入的内容自动转成数字类型。

- trim修饰符：
  如果输入的内容首尾有很多空格，通常我们希望将其去除，trim修饰符可以过滤内容左右两边的空格。
  
  
  
  ```html
  <div id="app">
    <!--1.修饰符: lazy-->
    <input type="text" v-model.lazy="message">
    <h2>{{message}}</h2>
  
    <!--2.修饰符: number-->
    <input type="number" v-model.number="age">
    <h2>{{age}}-{{typeof age}}</h2>
  
    <!--3.修饰符: trim-->
    <input type="text" v-model.trim="name">
    <h2>您输入的名字:{{name}}</h2>
  </div>
  
  <script src="../js/vue.js"></script>
  <script>
    const app = new Vue({
      el: '#app',
      data: {
        message: '你好啊',
        age: 0,
        name: ''
      }
    })
  </script>
  ```
  
  ![image-20201221162540806](/images/vue-02/image-20201221162540806.png)

### 3、事件监听指令

#### 3.1 v-on指令

在前端开发中，我们需要经常和用于交互。这个时候，我们就必须监听用户发生的时间，比如点击、拖拽、键盘事件等等
在Vue中如何监听事件呢？使用v-on指令。

##### 3.1.1 v-on:click 基本使用

这里写一个简单的栗子：

```vue
<body>

<div id="app">
  <h2>{{counter}}</h2>
  <button @click="increment">+</button>
  <button @click="decrement">-</button>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      counter: 0
    },
    methods: {
      increment() {
        this.counter++
      },
      decrement() {
        this.counter--
      }
    }
  })
</script>

</body>
```

1. 定义数据counter，用于表示计数器数字，初始值设置为0
2. v-on:click 表示当发生点击事件的时候，触发等号里面的**表达式**或者**函数**
3. 表达式`counter++`和`counter--`分别实现计数器数值的加1和减1操作
4. 语法糖：我们可以将`v-on:click`简写为`@click`

**另外**，当通过methods中定义方法，以供@click调用时，需要注意参数问题：
情况一：如果该方法不需要额外参数，那么方法后的()可以不添加。但是注意：如果方法本身中有一个参数，那么会默认将原生事件event参数传递进去
情况二：如果需要同时传入某个参数，同时需要event时，可以通过$event传入事件。

```vue
<div id="app">
  <!--1.事件调用的方法没有参数-->
  <button @click="btn1Click()">按钮1</button>
  <button @click="btn1Click">按钮1</button>

  <!--2.在事件定义时, 写方法时省略了小括号, 但是方法本身是需要一个参数的, 这个时候, Vue会默认将浏览器生产的event事件对象作为参数传入到方法-->
  <!--<button @click="btn2Click(123)">按钮2</button>-->
  <!--<button @click="btn2Click()">按钮2</button>-->
  <button @click="btn2Click">按钮2</button>

  <!--3.方法定义时, 我们需要event对象, 同时又需要其他参数-->
  <!-- 在调用方式, 如何手动的获取到浏览器参数的event对象: $event-->
  <button @click="btn3Click(abc, $event)">按钮3</button>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      abc: 123
    },
    methods: {
      btn1Click() {
        console.log("btn1Click");
      },
      btn2Click(event) {
        console.log('--------', event);
      },
      btn3Click(abc, event) {
        console.log('++++++++', abc, event);
      }
    }
  })

  // 如果函数需要参数,但是没有传入, 那么函数的形参为undefined
  // function abc(name) {
  //   console.log(name);
  // }
  //
  // abc()
</script>
```

![image-20201218141523006](/images/vue-02/image-20201218141523006.png)

#### 3.2 其他监听事件

前面我们介绍了v-on:click，除此之外，还有很多其他的可监听事件：

| 属性                                                         | 事件何时触发                         |
| :----------------------------------------------------------- | :----------------------------------- |
| [abort](https://www.w3school.com.cn/jsref/event_onabort.asp) | 图像的加载被中断。                   |
| [blur](https://www.w3school.com.cn/jsref/event_onblur.asp)   | 元素失去焦点。                       |
| [change](https://www.w3school.com.cn/jsref/event_onchange.asp) | 域的内容被改变。                     |
| [click](https://www.w3school.com.cn/jsref/event_onclick.asp) | 当用户点击某个对象时调用的事件句柄。 |
| [dblclick](https://www.w3school.com.cn/jsref/event_ondblclick.asp) | 当用户双击某个对象时调用的事件句柄。 |
| [error](https://www.w3school.com.cn/jsref/event_onerror.asp) | 在加载文档或图像时发生错误。         |
| [focus](https://www.w3school.com.cn/jsref/event_onfocus.asp) | 元素获得焦点。                       |
| [keydown](https://www.w3school.com.cn/jsref/event_onkeydown.asp) | 某个键盘按键被按下。                 |
| [keypress](https://www.w3school.com.cn/jsref/event_onkeypress.asp) | 某个键盘按键被按下并松开。           |
| [keyup](https://www.w3school.com.cn/jsref/event_onkeyup.asp) | 某个键盘按键被松开。                 |
| [load](https://www.w3school.com.cn/jsref/event_onload.asp)   | 一张页面或一幅图像完成加载。         |
| [mousedown](https://www.w3school.com.cn/jsref/event_onmousedown.asp) | 鼠标按钮被按下。                     |
| [mousemove](https://www.w3school.com.cn/jsref/event_onmousemove.asp) | 鼠标被移动。                         |
| [mouseout](https://www.w3school.com.cn/jsref/event_onmouseout.asp) | 鼠标从某元素移开。                   |
| [mouseover](https://www.w3school.com.cn/jsref/event_onmouseover.asp) | 鼠标移到某元素之上。                 |
| [mouseup](https://www.w3school.com.cn/jsref/event_onmouseup.asp) | 鼠标按键被松开。                     |
| [reset](https://www.w3school.com.cn/jsref/event_onreset.asp) | 重置按钮被点击。                     |
| [resize](https://www.w3school.com.cn/jsref/event_onresize.asp) | 窗口或框架被重新调整大小。           |
| [select](https://www.w3school.com.cn/jsref/event_onselect.asp) | 文本被选中。                         |
| [submit](https://www.w3school.com.cn/jsref/event_onsubmit.asp) | 确认按钮被点击。                     |
| [unload](https://www.w3school.com.cn/jsref/event_onunload.asp) | 用户退出页面。                       |

#### 3.3 按键监听修饰符

![image-20201218141806612](/images/vue-02/image-20201218141806612.png)

- stop修饰符，可以阻止事件向上级标签的冒泡行为。**子元素**加上stop修饰符号之后，点击子元素div，只有childMethod方法被触发，parentMethod方法不会被触发。

- self修饰符，表示被该修饰符修饰的父元素不接收子元素的事件冒泡行为。**父元素**加上self修饰符之后，点击子元素div，只有childMethod方法被触发，parentMethod方法不会被触发。
- prevent修饰符，可以阻止一些html标签的默认行为。
  - 比如：`<a>`标签的默认行为是，实现跳转
  - 比如： `<input type="submit"/>`默认行为是提交表单
- enter修饰符（按键监听修饰符的一种），可以监听回车按键的操作。
- once修饰符，表示事件只可以被触发监听一次，以后再操作则无效。
- capture修饰符，表示开启事件传播的捕获模式，事件由父元素向子元素传播，较少用到

Vue给我们定义好了一些常用的按键监听修饰符，如下：

- `.enter` 监听回车键
- `.tab` 监听Tab键
- `.delete` (监听“删除”和“退格”键)
- `.esc` 监听ESC键
- `.space` 监听空格键
- `.up` 监听"上"键
- `.down` 监听"下"键
- `.left` 监听"左"键
- `.right` 监听"右"键

如果我们觉得上面的键盘监听修饰符不够用，我们还可以自定义，如下定义监听F1按键.

F1按键的键盘码是112。

```
// 可以使用 `v-on:keyup.f1`
Vue.config.keyCodes.f1 = 112
```

![image-20201218143955030](/images/vue-02/image-20201218143955030.png)



### 4、 条件判断指令

#### 4.1  v-if 、v-else-if 、v-else的使用方式

```html
<body>

<div id="app">
  <span v-if="isUser">
    <label for="username">用户账号</label>
    <input type="text" id="username" placeholder="用户账号">
  </span>
  <span v-else>
    <label for="email">用户邮箱</label>
    <input type="text" id="email" placeholder="用户邮箱">
  </span>
  <button @click="isUser = !isUser">切换类型</button>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      isUser: true
    }
  })
</script>
```

![image-20201218151220712](/images/vue-02/image-20201218151220712.png)

![image-20201218151258066](/images/vue-02/image-20201218151258066.png)

如果我们在有输入内容的情况下，虽然切换了类型，我们会发现文字依然显示之前的输入的内容。这是因为Vue在进行DOM渲染时，出于性能考虑，会尽可能的复用已经存在的元素，而不是重新创建新的元素。如果我们不希望Vue出现类似重复利用的问题，可以给对应的input添加key，并且我们需要保证key的不同。

![image-20201218151403383](/images/vue-02/image-20201218151403383.png)

这样就可以避免上面的问题。

#### 4.2 v-show

v-show的用法和v-if非常相似，也用于决定一个元素是否渲染，区别：

- v-if当条件为false时，压根不会有对应的元素在DOM中。

- v-show当条件为false时，仅仅是将元素的display属性设置为none而已。

开发中如何选择呢？

- 当需要在显示与隐藏之间切片很频繁时，使用v-show

- 当只有一次切换时，通过使用v-if

```html
<body>

<div id="app">
  <!--v-if: 当条件为false时, 包含v-if指令的元素, 根本就不会存在dom中-->
  <h2 v-if="isShow" id="aaa">{{message}}</h2>

  <!--v-show: 当条件为false时, v-show只是给我们的元素添加一个行内样式: display: none-->
  <h2 v-show="isShow" id="bbb">{{message}}</h2>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      message: '你好啊',
      isShow: true
    }
  })
</script>
```



### 5、循环遍历指令v-for

当我们有一组数据需要进行渲染时，我们就可以使用v-for来完成。

- 如果在遍历的过程中不需要使用索引值

  语法：v-for="movie in movies" ，依次从movies中取出movie，并且在元素的内容中，我们可以使用Mustache语法，来使用movie

- 如果在遍历的过程中，我们需要拿到元素在数组中的索引值呢？
  语法格式：v-for=(item, index) in items，其中的index就代表了取出的item在原数组的索引值。

比如某个对象中存储着你的个人信息，我们希望以列表的形式显示出来

#### 5.1 遍历数组

```html
<div id="app">
  <!--1.在遍历的过程中,没有使用索引值(下标值)-->
  <ul>
    <li v-for="item in names">{{item}}</li>
  </ul>

  <!--2.在遍历的过程中, 获取索引值-->
  <ul>
    <li v-for="(item, index) in names">
      {{index+1}}.{{item}}
    </li>
  </ul>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      names: ['why', 'kobe', 'james', 'curry']
    }
  })
</script>
```

![image-20201218153446715](/images/vue-02/image-20201218153446715.png)

#### 5.2 遍历对象

```html
<div id="app">
  <!--1.在遍历对象的过程中, 如果只是获取一个值, 那么获取到的是value-->
  <ul>
    <li v-for="item in info">{{item}}</li>
  </ul>


  <!--2.获取key和value 格式: (value, key) -->
  <ul>
    <li v-for="(value, key) in info">{{value}}-{{key}}</li>
  </ul>


  <!--3.获取key和value和index 格式: (value, key, index) -->
  <ul>
    <li v-for="(value, key, index) in info">{{value}}-{{key}}-{{index}}</li>
  </ul>
</div>

<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      info: {
        name: 'why',
        age: 18,
        height: 1.88
      }
    }
  })
</script>
```

![image-20201218153654477](/images/vue-02/image-20201218153654477.png)

#### 5.3 v-for的key属性

官方推荐我们在使用v-for时，给对应的元素或组件添加上一个:key属性。我们先看一个栗子。

![image-20201218154840393](/images/vue-02/image-20201218154840393.png)

![](/images/vue-02/c8ad76ae7ec1348fa7d6bfe9f1615d80_400x126.gif)

- 我们首先勾选了curry
- 然后输入“杜兰特”，回车。杜兰特作为一个对象加入了数组，并显示在页面上
- 但是，bug出现了，我们本来勾选的库里，现在变成勾选杜兰特。

以上的Bug导致的原因，是VUE本身的虚拟dom渲染的算法决定的，这个算法叫做：Diff算法。所以我们需要使用key来给每个节点做一个唯一标识。Diff算法就可以正确的识别此节点找到正确的位置区插入新的节点。

通常就是该数组元素对应的数据库记录的主键作为key。将本文中的代码做如下修改，即可解决上文中提到的bug。

```html
<li v-for="item in players"  v-bind:key="item.id">
```

### 