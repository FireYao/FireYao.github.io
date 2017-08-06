---
title: Java8 新特性 Lambda表达式
date: 2017-08-05 13:18:27
tags: [Java,Java8,Lambda]
toc: true
reward: false
---

### 什么是Lambda表达式

  **Lambda**可以理解为是一个**匿名函数**，Lambda表达式可以说是一段可以传递的代码。可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使Java的语言表达能力得到了提升。

  在Java8以前，我们是通过接口来传递代码的(面向接口的编程)。
<!-- more -->
  比如：

```java
 Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("run----");
            }
        };

 Thread th = new Thread(runnable);
```

这里Thread类需要的其实并不是Runnable对象，而是它的方法

```java
public abstract void run();
```

但是没有办法直接传递方法，只能通过接口来传递。

**Java8**提供了一种新的紧凑的传递代码的语法--就是**Lambda**表达式。比如刚才的Thread可以用lambda表达式修改为：

```java
Runnable runnable = (() -> System.out.println("run----"));
Thread th = new Thread(runnable);
//还可以继续简化
Thread t = new Thread(() -> System.out.println("run----"));
```

是不是简洁多了？。


#### 1.Lambda表达式语法

Lambda 表达式在Java 语言中引入了一个新的语法元素和操作符。这个操作符为 `->` ， 该操作符被称为 Lambda 操作符或剪头操作符。它将 Lambda 分为两个部分：

**左侧:** 指定了 Lambda 表达式所需要的所有参数

**右侧:**指定了 Lambda 体，即 Lambda 表达式要执行的功能即需传递的方法的实现。

**语法格式:**

**1). 无参,无返回值,Lambda体只需一条语句**

````java
Runnable runnable = (() -> System.out.println("run----"));
````
**2). Lambda需要一个参数**

````java
Consumer<String> consumer = ((str) -> System.out.println(str));
//只有一个参数时，参数小括号可以省略，如下
Consumer<String> consumer = (str -> System.out.println(str));
````
**3). Lambda  需要两个参数，并且有返回值**

````java
Comparator<Integer> comparator = ((num1, num2) -> {
   return num1 - num2;
});
//当表达式内只有一条语句时，return和大括号可以省略，如下
Comparator<Integer> comparator = ((num1, num2) -> num1 - num2);
````
**4). 数据类型可以省略，因为可由编译器推断得出，称为“类型推断”**

````java
BinaryOperator<Long> binaryOperator = ((Long num1, Long num2) -> num1 + num2);
//(Long l1, Long l2) 中参数类型可以省略，编译器可以自动推断，如下↓
BinaryOperator<Long> binaryOperator = ((num1, num2) -> num1 + num2);
````

**可以看出，相比匿名内部类，传递代码变得更为直观，不再有实现接口的模板代码，不再声明方法，也名字也没有，而是直接给出了方法的实现代码**

#### **2.变量引用**

​	与匿名内部类类似，Lambda表达式也可以访问定义在主体代码外部的变量，但对于局部变量，它也只能访问final类型的变量，与匿名内部类的区别是，它不要求变量声明为final，但变量事实上不能被重新赋值。比如：

````java
Integer num = 1;
Function<Integer, Integer> function = (integer -> num);
````

可以访问局部变量num，但num不能被重新赋值，如果这样写：

````java
Integer num = 1;
Function<Integer, Integer> function = (integer -> num++);
//编译器报错
//Variable used in lambda expression should be final or effectively final
````

### 与匿名内部内比较

从以上内容可以看出，Lambda表达式与匿名内部类很像，主要就是简化了语法，使得编写更加简单，但Lambda与匿名内部类不同的是，java会为每个匿名内部类生成一个一个类，而Lambda表达式不会。

Lambda表达式不是匿名内部类，那它的类型到底是什么呢？是**函数式接口**。

### 小结

​	本章介绍了Lambda表达式，Lambda语法以及在Lambda表达式中的变量引用。

下篇将介绍Java8新特性，**函数式接口**


