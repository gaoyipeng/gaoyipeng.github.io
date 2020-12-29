---
title: Vue过滤器的使用
date: 2020-12-29 14:02:06
tags: Vue
categories: Vue
description: Vue过滤器的使用
typora-root-url: ..
---

Vue.js 允许自定义过滤器，可被用于一些常见的文本格式化。过滤器可以用在两个地方：**双花括号插值和 `v-bind` 表达式** (后者从 2.1.0+ 开始支持)。过滤器应该被添加在 JavaScript 表达式的尾部，由“管道”符号指示：

```html
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>
```

过滤器可以分为**全局过滤器**和**本地过滤器**，当全局过滤器和局部过滤器重名时，会采用局部过滤器。

```html
<body>
<div id="app">
    全局过滤器：{{message | capitalizeGlobal}}
    <br/>
    本地过滤器: {{message | capitalize}}
</div>

</body>
<script src="../js/vue.js"></script>
<script>
    //全局过滤器
    Vue.filter('capitalizeGlobal', function (value) {
        if (!value) return ''
        value = value.toString()
        return value.charAt(1).toUpperCase() + value.slice(1)
    })

    const app = new Vue({
        el: '#app', // 用于挂载要管理的元素
        data: { // 定义数据
            message: 'hello'
        },
        //本地过滤器
        filters: {
            capitalize: function (value) {
                if (!value) return ''
                value = value.toString()
                return value.charAt(0).toUpperCase() + value.slice(1)
            }
        }
    })
</script>
```

![image-20201229141201885](/images/vue-05/image-20201229141201885.png)

过滤器可以串联：

```
{{ message | filterA | filterB }}
```

在这个例子中，`filterA` 被定义为接收单个参数的过滤器函数，表达式 `message` 的值将作为参数传入到函数中。然后继续调用同样被定义为接收单个参数的过滤器函数 `filterB`，将 `filterA` 的结果传递到 `filterB` 中。

过滤器是 JavaScript 函数，因此可以接收参数：

```
{{ message | filterA('arg1', arg2) }}
```

这里，`filterA` 被定义为接收三个参数的过滤器函数。其中 `message` 的值作为第一个参数，普通字符串 `'arg1'` 作为第二个参数，表达式 `arg2` 的值作为第三个参数。

**工程化项目中(一般使用Cli脚手架搭建)，全局过滤器的使用方法如下：**

在项目中使用到的经常用到过滤器，比如时间，数据截取等过滤器，如果在每个.vue中都可以复制同一个过滤器，这可以达到目的，但是遇到方法有bug时就需要诸葛修改进入不同的页面修改，这样既费时又费力，优先可以考虑注册全局的过滤器。

 定义方法如下：

新建filters/index.js

```js
const isNullOrEmpty = function(val) {
    if (val == null || val == "" || typeof(val) == undefined) {
        return true;
    } else {
        return false;
    }
}
 
const timeFormat = (value, format) => {
    let date = new Date(value);
    let y = date.getFullYear();
    let m = date.getMonth() + 1;
    let d = date.getDate();
    let h = date.getHours();
    let min = date.getMinutes();
    let s = date.getSeconds();
    let result = "";
    if (format == undefined) {
        result = `${y}-${m < 10 ? "0" + m : m}-${d < 10 ? "0" + d : d} ${
        h < 10 ? "0" + h : h
      }:${min < 10 ? "0" + min : min}:${s < 10 ? "0" + s : s}`;
    }
    if (format == "yyyy-mm-dd") {
        result = `${y}-${m < 10 ? "0" + m : m}-${d < 10 ? "0" + d : d}`;
    }
    if (format == "yyyy-mm") {
        result = `${y}-${m < 10 ? "0" + m : m}`;
    }
    if (format == "mm-dd") {
        result = ` ${mm < 10 ? "0" + mm : mm}:${ddmin < 10 ? "0" + dd : dd}`;
    }
    if (format == "hh:mm") {
        result = ` ${h < 10 ? "0" + h : h}:${min < 10 ? "0" + min : min}`;
    }
    if (format == "yyyy") {
        result = `${y}`;
    }
    return result;
};
 
 
export {
    isNullOrEmpty,
    timeFormat
}
```

　在main.js中引入和注册全局过滤器

```js
import * as filters from '../filters/index'
Object.keys(filters).forEach(key => {
    Vue.filter(key, filters[key])
})
```

此时就可以在不同的.vue中使用定义的全局过滤器了

```html
{{date|isNullOrEmpty}}是否为空<br/>
{{date|timeFormat('yyyy-mm-dd')}} 时间过滤器<br>
{{date|timeFormat('yyyy-mm')}} 时间过滤器yyyy-mm<br>
{{date|timeFormat('hh:mm')}} 时间过滤器hh:mm<br>
{{date|timeFormat('yyyy')}} 时间过滤器yyyy<br>
{{date|timeFormat}} 时间过滤器yyyy<br>
```

效果如下：

![img](https://img2018.cnblogs.com/blog/1364613/201903/1364613-20190326200339036-125787923.png)