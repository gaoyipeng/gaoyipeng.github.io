---
title: SpringMVC @ModelAttribute注解的用法
date: 2020-03-15 22:13:33
tags: SpringMVC
categories: SpringMVC
description: SpringMVC @ModelAttribute注解的用法
typora-root-url: ..
---
@ModelAttribute主要的作用是将数据添加到对象模型中，在视图页面展示时使用。
@ModelAttribute 等价于 model.addAttribute("abc", "abc"); 或 map.put("abc","abc")

## 小知识点

首先我们需要知道，在SpringMVC中，以下3种写法效果相同。

```
    @RequestMapping(value = "/index1")
    public String index(Dog dog) {
        dog.setName("二哈1");
        dog.setAge(1);
        return "index";
    }

    @RequestMapping(value = "/index2")
    public String index1( Model model) {
        Dog dog = new Dog();
        dog.setName("二哈2");
        dog.setAge(2);
        model.addAttribute("dog",dog);
        return "index";
    }

    @RequestMapping(value = "/index3")
    public String index2( Map map) {
        Dog dog = new Dog();
        dog.setName("二哈3");
        dog.setAge(3);
        map.put("dog",dog);
        return "index";
    }
```
index.jsp：
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    String path = request.getContextPath();
    String basePath = request.getScheme() + "://"
            + request.getServerName() + ":" + request.getServerPort()
            + path + "/";
%>
<html>
<head>
    <title>Index</title>
    <link rel="stylesheet" type="text/css" href="<%=basePath%>statics/css/index.css">
</head>
<body>
<h2>Test success!</h2>
dog:${requestScope.dog}
</body>
</html>
```
分别访问http://localhost:8080/springmvc/index1
![dog1](/images/springmvc/dog1.png)
http://localhost:8080/springmvc/index2
![dog2](/images/springmvc/dog2.png)
http://localhost:8080/springmvc/index3
![dog3](/images/springmvc/dog3.png)

## @ModelAttribute修饰一个方法

先贴出一个实体类Dog:
```
public class Dog {
    private String name;
    private Integer age;
    public Dog() {
    }
    public Dog(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
  ...getter,setter

}

```
<strong>以下示例1，2，3类似，被@ModelAttribute注释的方法会在此controller每个方法执行前被执行，因此对于一个Controller映射有多个URL的用法来说，要谨慎使用。
</strong>



### 例1： @ModelAttribute标注在 返回值为void的方法上
```
@Controller
public class DemoController {

    @ModelAttribute
    public void testModelAttribute(Model model){
        model.addAttribute("attribute","attribute");
    }

    @RequestMapping(value = "/index1")
    public String index(Dog dog) {
        dog.setName("二哈1");
        dog.setAge(1);
        return "index";
    }
}
```
index.jsp:
```
<body>
<h2>Test success!</h2>

dog:${requestScope.dog}

<br>

attribute:${requestScope.attribute}

</body>
```

![model1](/images/springmvc/model1.png)

说明：testModelAttribute方法在 index 方法之前先被调用，它在model中添加了("attribute","attribute"),在它执行后 index 被调用，返回视图index和model,此时的model中包含2个k-v键值对。

<font color="red">注意:</font>如果我们修改testModelAttribute和index方法如下:

```
@ModelAttribute
    public void testModelAttribute(Model model){
        Dog dog = new Dog("德牧",2);
        model.addAttribute("dog",dog);
        //model.addAttribute("attribute","attribute");
    }

@RequestMapping(value = "/index1")
public String index(Dog dog) {
    dog.setAge(1);
    return "index";
}
```
![model1-1.png](/images/springmvc/model1-1.png)

<font color="red">结论：方法返回的model值相同（inde方法model为形参Dog首字母小写），均为dog。那么。testModelAttribute方法产生的model中的dog中的属性值，会覆盖掉index方法产生的model中的dog中的为null的属性值。</font>

### 例2： @ModelAttribute标注在 返回具体类的方法

```
    @ModelAttribute
    public Dog testModelAttribute(){
        Dog dog = new Dog("德牧",3);
        return dog;
    }

    @RequestMapping(value = "/index1")
    public String index() {
        return "index";
    }
```
![model2.png](/images/springmvc/model2.png)

说明：这种情况，model属性的名称没有指定，它由返回类型隐含表示，如这个方法返回Dog类型，那么这个model属性的名称是dog(首字母小写)。

### 例3： @ModelAttribute(value="")注释返回具体类的方法，但是指明了value值

```
    @ModelAttribute(value = "attribute")
    public Dog testModelAttribute(){
        Dog dog = new Dog("德牧",2);
        return dog;
    }

    @RequestMapping(value = "/index1")
    public String index(Dog dog) {
        dog.setName("二哈1");
        dog.setAge(1);
        return "index";
    }
```

![model3.png](/images/springmvc/model3.png)

说明：这个例子中使用@ModelAttribute注释的value属性，来指定model属性的名称。model属性对象就是方法的返回值。

### 例4： @ModelAttribute和@RequestMapping同时注释一个方法

```
    @ModelAttribute
    public  Dog  testModelAttribute(){
        Dog dog = new Dog("德牧",2);
        return dog;
    }

    @RequestMapping(value = "/index")
    @ModelAttribute(value = "attribute")
    public String index() {
        return "attributeValue";
    }

```

![model4.png](/images/springmvc/model4.png)

说明：index()这个方法的返回值并不是表示一个视图名称，而是model属性的值，视图名称由RequestToViewNameTranslator根据请求"/index"转换为逻辑视图index。
   Model属性名称有@ModelAttribute(value="")指定，相当于在request中封装了key="attribute"，value="attributeValue"。

## @ModelAttribute注解修饰一个方法的参数

```
  @ModelAttribute(value = "dog")
    public Dog testModelAttribute(){
        Dog dog = new Dog("德牧",2);
        return dog;
    }

    @RequestMapping(value = "/index1")
    public String index(@ModelAttribute(value = "dog") Dog dog) {
        dog.setName("二哈");
        return "index";
    }
```
![param1.png](/images/springmvc/param1.png)

说明：在这个例子里，@ModelAttribute(value = "dog") Dog dog 注释方法参数，参数dog的值来源于testModelAttribute()方法中的model属性。
   　　此时如果方法体没有标注@SessionAttributes("dog")，那么scope为request，如果标注了，那么scope为session

## @ModelAttribute注解修饰一个方法的返回值

```
    @RequestMapping(value = "/index")
    public @ModelAttribute(value = "dog2") Dog  testModelAttribute(){
        Dog dog = new Dog("德牧",2);
        return dog;
    }
```
index.jsp
```
<body>
<h2>Test success!</h2>

dog2:${requestScope.dog2}

</body>
```
![return1.png](/images/springmvc/return1.png)

说明： @ModelAttribute(value = "dog2")，会添加返回值到model（ 名字为user2 ） 中供视图展示使用。

OK,都介绍完了，有点绕，多看几遍就好了。

参考：https://blog.csdn.net/leo3070/article/details/81046383
