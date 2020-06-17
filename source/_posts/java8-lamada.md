---
title: Java8 Lambda表达式
date: 2019-12-29 18:32:12
tags: Java8
categories: Java8
description: Java8 Lambda表达式
typora-root-url: ..
---

{% note info %} Java 8的Lambda表达式，简化了匿名函数的表达方式。Lambda表达式可以直接以内联的形式为函数式接口的抽象方法提供实现，并把整个表达式作为函数式接口的实例。

什么是函数式接口？简单来说就是只包含一个抽象方法的接口，允许有默认的实现（使用default关键字描述方法）。函数式接口建议使用@FunctionalInterface注解标注，虽然这不是必须的，但是这样做更符合规范。{% endnote %}

本节主要记录java8 内置函数式接口的使用。

## java8 中常用的函数式接口

|函数式接口|函数描述符|说明|
|:---:|:---:|:---:|
|Predicate  |T->boolean  |断言：主要作用就是用于判断作用 |
|Consumer   |T->void  |消费者：该函数式接口用于消费一个对象，即接收一个对象，对其执行某些操作，然后没有返回值。 |
|Supplier   |()->T  |供应商：无参数，返回T类型的对象 |
|Function  |T->R  |它接受一个泛型T的对象，并返回一个泛型R的对象 |

## Predicate 断言

源码如下：
```
package java.util.function;
import java.util.Objects;
/**
 * @since 1.8
 */
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```
java.util.function.Predicate<T>接口定义了一个名叫test的抽象方法，它接受泛型T对象，并返回一个boolean。
除了抽象方法外，java.util.function.Predicate<T>接口还定义了三个默认方法：and，negate和or，对应“与”，“非”和“或”操作，这样我们便可以复合Lambda表达式了.

示例代码：
```
public static void main(String[] args) {
    /**
     * 函数式接口：Predicate
     */
    Predicate<Integer> isEven = (i) -> i%2 ==0 ;
    //判断： 是否为偶数
    System.out.println( isEven.test(18) );
    //判断： 大于10 且 为偶数
    System.out.println( isEven.and((i) -> i>10).test(18) );
    //判断： 小于10 或 为偶数
    System.out.println( isEven.or((i) -> i<10).test(18) );
    //判断： 奇数判断
    System.out.println( isEven.negate().test(18) );
}
```
运行结果为：
true 
true 
true 
false

## Consumer 消费者，接收参数，无返回值

源码如下：
```
package java.util.function;
import java.util.Objects;

@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```
java.util.function.Consumer<T>定义了一个名叫accept的抽象方法，它接受泛型T的对象，没有返回(void)，函数描述符为(T) -> void。
其还提供了一个默认方法andThen

示例代码：
```
public static void main(String[] args) {
    /**
     * 函数式接口：Consumer
     */
    String str = "hello";
    Consumer<String> consumer = (i) -> System.out.println(i);
    //判断： 输出 "hello"
    consumer.accept(str);
    //判断： 输出"hello" 和 "hello world"
    consumer.andThen((i) -> System.out.println(i+" world")).accept(str);
}
```

## Supplier  供应商：无参数，返回T类型的对象 

源码如下：
```
package java.util.function;
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

java.util.function.Supplier<T>很简单，只定义了一个名叫get的抽象方法，它不接收参数，返回泛型T的对象，函数描述符为() -> T

示例代码：
```
public static void main(String[] args) {
    /**
     * 函数式接口：Supplier
     */
    Supplier<String> supplier = () -> new String("supplier");
    System.out.println(supplier.get());// 返回一个String对象"supplier"
}
```

## Function 

源码如下：
```
package java.util.function;
import java.util.Objects;
/**
 * @since 1.8
 */
@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}

```

java.util.function.Function<T, R>接口定义了一个叫作apply的方法，它接受一个泛型T的对象，并返回一个泛型R的对象，函数描述符为(T) -> R 
除了抽象方法外，java.util.function.Function<T, R>接口还定义了三个默认方法：compose，andThen,identity

示例代码：
```
public static void main(String[] args) {
    /**
     * 函数式接口：Function
     */
    // 接收一个Integer ,返回一个String
    Function<Integer,String> function = (i)-> {
        return Integer.toString(i);
    };
    //接收 integer 123 ，返回String "123"
    System.out.println(function.apply(123));
    System.out.println(function.apply(123) instanceof String);

    //compose 过程为：f(g(2))，也就是 (2*2)+1 ,即把g的执行结果当做f的参数
    Function<Integer, Integer> f = (x) -> x + 1;
    Function<Integer, Integer> g = (x) -> x * 2;
    f.compose(g).apply(2); // 5

    //andThen 过程为：g1(f1(2))，也就是(2+1)*2 ，即把f1的执行结果当做g1的参数
    Function<Integer, Integer> f1 = (x) -> x + 1;
    Function<Integer, Integer> g1 = (x) -> x * 2;
    f1.andThen(g1).apply(2); // 6

}
```
## 最后附上全部代码

```
package com.sxdx.spring.security.oauth2.controller;

import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

public class Java8TestController {

    public static void main1(String[] args) {
        /**
         * 函数式接口：Predicate
         */
        Predicate<Integer> isEven = (i) -> i%2 ==0 ;
        //判断： 是否为偶数
        System.out.println( isEven.test(18) );
        //判断： 大于10 且 为偶数
        System.out.println( isEven.and((i) -> i>10).test(18) );
        //判断： 小于10 或 为偶数
        System.out.println( isEven.or((i) -> i<10).test(18) );
        //判断： 奇数判断
        System.out.println( isEven.negate().test(18) );
    }
    public static void main2(String[] args) {
        /**
         * 函数式接口：Consumer
         */
        String str = "hello";
        Consumer<String> consumer = (i) -> System.out.println(i);
        //判断： 输出 "hello"
        consumer.accept(str);
        //判断： 输出"hello" 和 "hello world"
        consumer.andThen((i) -> System.out.println(i+" world")).accept(str);
    }

    public static void main3(String[] args) {
        /**
         * 函数式接口：Supplier
         */
        // 无参，直接返回一个String字符串
        Supplier<String> supplier = () -> new String("supplier");
        System.out.println(supplier.get());
    }

    public static void main(String[] args) {
        /**
         * 函数式接口：Function
         */
        // 接收一个Integer ,返回一个String
        Function<Integer,String> function = (i)-> {
            return Integer.toString(i);
        };
        //接收 integer 123 ，返回String "123"
        System.out.println(function.apply(123));
        System.out.println(function.apply(123) instanceof String);

        //compose 过程为：f(g(2))，也就是 (2*2)+1 ,即把g的执行结果当做f的参数
        Function<Integer, Integer> f = (x) -> x + 1;
        Function<Integer, Integer> g = (x) -> x * 2;
        f.compose(g).apply(2); // 5

        //andThen 过程为：g1(f1(2))，也就是(2+1)*2 ，即把f1的执行结果当做g1的参数
        Function<Integer, Integer> f1 = (x) -> x + 1;
        Function<Integer, Integer> g1 = (x) -> x * 2;
        f1.andThen(g1).apply(2); // 6

    }
}

```

参考文章：https://mrbird.cc/java8lambda2.html
         https://www.runoob.com/java/java8-lambda-expressions.html