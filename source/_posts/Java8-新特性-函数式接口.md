---
title: Java8 新特性 函数式接口
date: 2017-08-06 11:55:03
categories:
- Java
tags: [Java,Java8,函数式接口]
toc: true
reward: false
---

### 什么是函数式接口

​	**Java 8引入了函数式接口的概念**

​	1). 只包含一个抽象方法的接口，称为**函数式接口**

​	2). 函数式接口可以被隐式转换为lambda表达式。

​	3). 在任意函数式接口上使用@FunctionalInterface注解，这样做可以检查它是否是一个函数式接口，同时javadoc 也会包含一条声明，说明这个接口是一个函数式接口。

<!-- more -->

### 预定义的函数式接口

​	Java 8定义了大量的预定义函数式接口，用于常见类型的代码传递，这些函数定义在包java.util.function下，

其中有四大核心函数式接口。

|         函数式接口         | 参数类型 |  返回类型   |                 用途                  |
| :-------------------: | :--: | :-----: | :---------------------------------: |
|  Consumer<T>(消费型接口)   |  T   |  void   |    对类型为T的对象应用操作。void accept(T t)    |
|  Supplier<T>(供给型接口)   |  无   |    T    |         返回类型为T的对象。 T get();         |
| Function<T, R>(函数型接口) |  T   |    R    | 对类型为T的对象应用操作并返回R类型的对象。R apply(T t); |
|  Predicate<T>(断言型接口)  |  T   | boolean | 确定类型为T的对象是否满足约束。boolean test(T t);  |

**Consumer<T> 消费型接口**

```java
public static void consume(double money, Consumer<Double> con){
	con.accept(money);
}
public static void main(String[] args) {
	consume(10000, (m) -> {
    	System.out.println("今日全场8折");
        System.out.println("顾客消费：" + (m * 0.8) + "元");
    });
}
```
**Supplier<T> 供给型接口**

```java
//生成num个整数,并存入集合
public List<Integer> getNumList(int num, Supplier<Integer> sup) {
	List<Integer> list = new ArrayList<>();
		for (int i = 0; i < num; i++) {
			Integer n = sup.get();
			list.add(n);
		}
	return list;
}

public static void main(String[] args) {
	//10个100以内的随机数
	List<Integer> numList = getNumList(10, () -> (int) (Math.random() * 100));
  	for (Integer num : numList) {
    	System.out.println(num);
    }
}
```
**Function<T, R> 函数型接口**

```java
/*
	Function接口常用于数据的处理转换,比如给定一个员工列表,需要返回名称列表
*/
public class Employee {
	private int id;
	private String name;
 	private double salary;
  	public Employee(String name){
		this.name = name;
  	}
  	public Employee(String name,double salary) {
		this.name = name;
		this.salary = salary;
	}
  	//省略getter setter
}
public class TestEmp{

	public static <T, R>List<R> map(List<T> list,Function<T, R> fun){
    	List<R> returnList = new ArrayList<>(list.size());
    	for (T e : list) {
			returnList.add(fun.apply(e));
      	}
    	return returnList
	}

	public static void main(String[] args) {
		List<Employee> employees = Arrays.asList(new Employee("老张"),
				new Employee("小李"),
        		new Employee("老王"),
                new Employee("小刘"),
                new Employee("小胖"));
		List<String> nameList = map(employees, (employee -> employee.getName()));
		System.out.println(nameList);
      	/*
      		console:[老张, 小李, 老王, 小刘, 小胖]
      	*/
    }
}
```
**Predicate<T> 断言型接口**

```java
public static <E> List<E> filter(List<E> list, Predicate<E> pred) {
	List<E> retList = new ArrayList<>();
    	for (E e : list) {
        	if (pred.test(e)) {
                retList.add(e);
            }
        }
	return retList;
}
public static void main(String[] args) {
		List<Employee> employees = Arrays.asList(new Employee("老张"),
    			new Employee("小李", 3000.00),
                new Employee("老王", 5000.00),
                new Employee("小刘", 7000.00),
                new Employee("小胖", 10000.00));
  		//过滤薪资小于5000的员工
      	List<Employee> filter = filter(employees,
                                       employee -> employee.getSalary() > 5000.00);
        for (Employee employee : filter) {
            System.out.println(employee.getName() + ":" + employee.getSalary());
        }
      	/*
      		console:小刘:7000.0
      				小胖:10000.0
      	*/
}
```
### 方法引用

​	当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用！方法引用：使用操作符 `::`将方法名和对象或类的名字分隔开来。如下三种主要使用情况 ：

**对象 : : 实例方法**

**类 : : 静态方法**

**类 : : 实例方法**

**基本用法**

例如：

```java
//静态方法
BinaryOperator<Double> binaryOperator = (x, y) -> Math.pow(x, y);
//等价于
BinaryOperator<Double> binaryOperator = Math::pow;
```

```java
//实例方法: 类::实例方法
Function<Employee, String> f = (Employee e) -> e.getName();
//等价于
Function<Employee, String> f = Employee::getName;
//---------------------------------------------------------
//对象::实例方法
Employee e = new Employee("小李", 3000.00);
Supplier<String> s = () -> e.getName();
//等价于↓
Supplier<String> s = e::getName;

```

### 构造方法

​	与函数式接口相结合，自动与函数式接口中方法兼容。可以把构造器引用赋值给定义的方法，与构造器参数
列表要与接口中抽象方法的参数列表一致！对于构造方法，方法引用的语法是<类名>::new，如`Employee::new`，如下语句：

```java
Function<String,Employee> f = (name)->new Employee(name);
//等价于↓
Function<String, Employee> f = Employee::new;
```

### 接口中的默认方法和静态方法

​	Java8以前，接口里的方法要求全部是抽象方法，Java8以后允许在接口里定义**默认方法**和**静态方法**,默认方法使用 **default** 关键字修饰。

例如：

```java
public interface MyFunction{
  void func();
  //声明一个接口的默认方法
  default void testDefalut(){
    System.out.println("MyFunction 默认方法");
  }
  //声明一个接口的静态方法
  static void testStatic(){
    System.out.println("MyFunction 静态方法");
  }
}
```

```java
//MyFunctionImpl实现接口MyFunction
public class MyFunctionImpl implements MyFunction {
    @Override
    public void func() {
        System.out.println("实现抽象方法");
    }

    public static void main(String[] args) {
        MyFunction my = new MyFunctionImpl();
        my.func();
        my.testDefalut();
        MyFunction.testStatic();
    }
  /*
      		实现抽象方法
            MyFunction 默认方法
            MyFunction 静态方法
  */
}
```

**默认方法的主要优势是提供一种拓展接口的方法，而不破坏现有代码。**

### 接口冲突

​	如果一个父接口提供一个默认方法，而另一个接口也提供了一个具有相同名称和参数列表的方法（不管方法是否是默认方法），那么必须覆盖该方法来解决冲突

```java
public interface AnotherFunction {
    default void testDefalut() {
        System.out.println("AnotherFunction 默认方法");
    }
}

public class FunctionImpl implements MyFunction,AnotherFunction{
  	@Override
    public void func() {
        System.out.println(" FunctionImpl 实现抽象方法");
    }

  	@Override
    public void testDefalut() {
		 System.out.println(" FunctionImpl 覆盖接口中默认方法解决冲突");
    }
}
```

​	如果不覆盖接口中相同的默认方法，那么`new MyFunctionImpl().testDefalut();`中调用的testDefalut方法到底是哪个接口的testDefalut()方法呢？所以必须在实现类中覆盖testDefalut()方法。

### 小结

​	本章中介绍了Java 8中的函数式接口，Java8四大核心函数式接口，方法的引用，接口的默认方法和静态方法。

下章将介绍Java8中强大的**Stream API**
