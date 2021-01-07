---
title: Vue (03) 数组的响应式方法
date: 2020-12-22 16:52:24
tags: Vue
categories: Vue
description: Vue (03) 数组的响应式方法
typora-root-url: ..
---

> 因为Vue是响应式的，所以当数据发生变化时，Vue会自动检测数据变化，视图会发生对应的更新。
>
> 对于数组，如果通过下标的方式修改数组中的内容，Vue无法测数据变化，则视图也不会随着数据的变化而变化。
>
> Vue中包含了一组观察数组编译的方法，使用它们改变数组也会触发视图的更新。

# 一、数组的响应式方法

先来看一个栗子：

```bash
<body>
<div id="app">
  <ul>
    <li v-for="item in letters">{{item}}</li>
  </ul>
  <button @click="btnClick">按钮</button>
</div>
<script src="../js/vue.js"></script>
<script>
  const app = new Vue({
    el: '#app',
    data: {
      letters: ['a', 'b', 'c', 'd']
    },
    methods: {
      btnClick() {
        // 注意: 通过索引值修改数组中的元素,Vue时无法监控到的
        this.letters[0] = 'bbbbbb';
        this.letters.splice(0, 1, 'bbbbbb')
      }
    }
  })
</script>
</body>
```

![image-20201222182444845](/images/vue-03/image-20201222182444845.png)

通过索引值修改数组中的元素,Vue时无法监控到的，必须使用下面的数组函数来操作。

## 1、push()

把一个或多个参数，并把它**推入**到数组的**末尾**。

```php
var array = [1,2,3];
array.push(4);
// 此时array的值为[1,2,3,4]
array.push(5,['cat','dog']);
// 此时array的值为[1,2,3,4,5,['cat','dog']]
```

## 2、pop()

**移除**数组**末尾**的元素并返回这个元素。

```php
var array = [1,2,3];
var num =array.pop();
//此时array为[1,2],num为3
```

## 3、shift()

**移出**数组中的**第一项**，并返回该移出的元素。

```php
var array = [1,2,3];
var num =array.shift();
//此时array为[2,3],num为1
```

## 4、unshift()

**移入**一个元素到数组的**头部**。

```php
var array = [2,3];
array.unshift(1);
//此时array为[1,2,3]
```

## 5、splice()

 移除数组内随意位置任意数量的连续的元素，并返回被删除的这些元素的数组。它主要有3个参数

| 参数              | 描述                                               |
| ----------------- | -------------------------------------------------- |
| index             | 必需，移除元素的起始位置，表示你要从该位置移除元素 |
| howmany           | 必需，移除元素的数量，如果为0，则不会移除元素      |
| item1, ..., itemX | 可选，向数组添加新的元素                           |

返回值

| 参数  | 描述                                 |
| ----- | ------------------------------------ |
| Array | 包含被删除项目的新数组，如果有的话。 |

注意：`splice()`是对数组进行直接修改！！！



```bash
let array = ["baby","boy","eat","apple","men","women"];
array.splice(2,2);//从数组第3个元素开始，删除2个元素
//现在array为["baby","boy","men","women"]


let array = ["baby","boy","eat","apple","men","women"];
let newArray = array.splice(2,2);//从数组第3个元素开始，删除2个元素
//newArray为["eat","apple"]

function htmlColorNames(arr) {
  arr.splice(0,2,'DarkSalmon','BlanchedAlmond')
  return arr;
} 
console.log(htmlColorNames(['DarkGoldenRod', 'WhiteSmoke', 'LavenderBlush', 'PaleTurqoise', 'FireBrick']));
//打印结果：DarkSalmon,BlanchedAlmond,LavenderBlush,PaleTurqoise,FireBrick
```

## 6、slice()

从数组里拷贝项目，并返回这些项目的一个新数组，它不会更改原来数组的项目，它有两个参数：

| 参数  | 描述                                                         |
| ----- | ------------------------------------------------------------ |
| start | (必填)开始拷贝元素的位置（索引）                             |
| end   | (可选)结束拷贝元素的位置（不包括本身），如果不指定该参数，那么就会拷贝从start到数组结尾的所有元素 |

```bash
let weatherConditions = ['rain', 'snow', 'sleet', 'hail', 'clear'];

let todaysWeather = weatherConditions.slice(1, 3);
// todaysWeather 等于 ['snow', 'sleet'];
// weatherConditions 仍然等于 ['rain', 'snow', 'sleet', 'hail', 'clear']
```

```bash
let forecast = ['cold', 'rainy', 'warm', 'sunny', 'cool', 'thunderstorms'];
let todayForecast = forecast.slice(4);
//todayForecast 等于['cool', 'thunderstorms']
```

## 7、sort()

按规则排序

```bash
let arr = ['d', 'c', 'b', 'a']
arr.sort(); 
// arr 等于  ['a', 'b', 'c', 'd']
```

## 8、reverse（）

反转数组

```bash
let arr = ['a', 'b', 'c', 'd']
arr.reverse(); 
// arr 等于  ['d', 'c', 'b', 'a']
```

# 二、数组的其他常用方法

**ES6 引入了许多有用的数组方法，例如：**

- `find()`，查找列表中的成员，返回 null 表示没找到
- `findIndex()`，查找列表成员的索引
- `some()`，检查某个断言是否至少在列表的一个成员上为真
- `includes`，列表是否包含某项

**下面的代码有助于你理解它们的用法：**

```js
const array = [{ id: 1, checked: true }, { id: 2 }];
arr.find(item => item.id === 2) // { id: 2 }
arr.findIndex(item => item.id === 2) // 1
arr.some(item => item.checked) // true

const numberArray = [1,2,3,4];
numberArray.includes(2) // true
Promises + Async/Await
```