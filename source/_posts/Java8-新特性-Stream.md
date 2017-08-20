---
title: Java8-新特性-Stream
date: 2017-08-20 14:09:12
categories:
- Java
tags: [Java,Java8,Stream]
toc: true
reward: false
---

### 了解Stream

​	Java8中有两个最为重要的改变，一个是Lambda表达式，另一个就是**Stream API**,针对常见的集合数据处理，Stream API 提供了一种高效且易于使用的数据处理方式。

### 什么是Stream

#### 基本概念

​	流(Stream)用于操作数据源所生成的元素序列。Java 8给Collection接口增加了两个默认方法，它们可以返回一个Stream

> default Stream<E> stream() {
> ​    return StreamSupport.stream(spliterator(), false);
> }//stream()返回的是一个顺序流
>
> default Stream<E> parallelStream() {
> ​    return StreamSupport.stream(spliterator(), true);
> }//parallelStream()返回的是一个并发流
>

1. **Stream 自己不会存储元素。**
2. **Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。**
3. **Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。**

<!-- more -->

####  基本示例

首先这里有一个Employee类

```java
public class Employee {
   private int id;
   private String name;
   private int age;
   private double salary;
   /*省略getter setter Constructor*/
}
```

```java
//Employee列表
List<Employee> emps = Arrays.asList(
      new Employee(102, "李四", 59, 6666.66),
      new Employee(101, "张三", 18, 9999.99),
      new Employee(103, "王五", 28, 3333.33),
      new Employee(104, "赵六", 20, 7777.77),
      new Employee(104, "赵六", 19, 7777.77),
      new Employee(104, "赵四", 40, 7777.77),
      new Employee(105, "田七", 38, 5555.55)
);
```

返回薪资大于5000的员工列表,java8以前是这样做的

```java
List<Employee> newEmps = new ArrayList<>();
for(Employee emp : emps){
  if(emp.salary > 5000.00){
    newEmps.add(emp);
  }
}
```

使用Stream API ,代码可以这样

```java
List<Employee> newEmps = emps.stream()
        .filter(s -> s.getSalary() > 5000.00)
        .collect(Collectors.toList());
```

先通过stream()得到一个Stream对象，然后调用Stream上的方法，filter()过滤得到薪资大于5000的,它的返回值依然是一个Stream,然后通过调用collect()方法并传递一个Collectors.toList()将结果集存放到一个List中.

使用Stream API处理集合类代码更加简洁易读.

下面介绍一下Stream中的两种操作

#### Stream的中间操作和终止操作

**中间操作**:

​	多个 中间操作可以连接起来形成一个 流水线，除非流水线上触发终止操作，否则 中间操作不会执行任何的 处理！而在 终止操作时一次性全部 处理，称为“惰性求值”

|           方法            |                   描述                   |
| :---------------------: | :------------------------------------: |
|   filter(Predicate p)   |         接收 Lambda ， 从流中排除某些元素。         |
|       distinct()        |  筛选，通过流所生成元素的 hashCode() 和 equals() 去  |
|   limit(long maxSize)   |            截断流，使其元素不超过给定数量。            |
|     map(Function f)     | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。 |
|   flatMap(Function f)   | 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流 |
| sorted(Comparator comp) |           产生一个新流，其中按比较器顺序排序            |
|        sorted()         |            产生一个新流，其中按自然顺序排序            |

**终止操作**:

​	终端操作会从流的流水线生成结果。其结果可以是任何不是流的值，例如：List、Integer，甚至是 void 。

|          方法          |                    描述                    |
| :------------------: | :--------------------------------------: |
| forEach(Consumer c)  |                   内部迭代                   |
| collect(Collector c) | 将流转换为其他形式。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法 |
|  max(Comparator c)   |                 返回流中最大值                  |
|  min(Comparator c)   |                 返回流中最小值                  |
|       count()        |                 返回流中元素总数                 |

**收集** : collect(Collector c)方法需要一个Collector 作为参数,Collector 接口中方法的实现决定了如何对流执行收集操作(如收集到 List、Set、Map)。Java8中提供了一个Collectors工具类, 工具中提供了很多静态方法，可以方便地创建常见收集器例

具体方法与实例如下表

|       方法       |         返回类型          |          作用          |
| :------------: | :-------------------: | :------------------: |
|     toList     |        List<T>        |     把流中元素收集到List     |
|     toSet      |        Set<T>         |     把流中元素收集到Set      |
|  toCollection  |     Collection<T>     |    把流中元素收集到创建的集合     |
|   groupingBy   |    Map<K, List<T>>    | 根据某属性值对流分组，属性为K，结果为V |
| partitioningBy | Map<Boolean, List<T>> |   根据true或false进行分区   |

这里只列出了一些常用的方法.具体参考Java8 Stream API

### Stream API 使用

#### 中间操作

1. 映射(map/flatMap)

   > map——接收 Lambda ， 将元素转换成其他形式或提取信息。接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
   >
   > `<R> Stream<R> map(Function<? super T, ? extends R> mapper);`
   >
   > map操作会将流里的每个元素按mapper转换后的结果(不管是) 添加到一个新流中.

   ```java
   List<String> list = Arrays.asList("1,2", "3,4");
   //每次mapper操作返回一个数组,将每个数组添加到新流中,最终生成Stream<String[]>
   Stream<String[]> stream = list.stream().map(s -> s.split(","));
   //每次mapper操作返回一个流Stream<String>,将每个流添加到新流中,最终生成Stream<Stream<String>>
   Stream<Stream<String>> streamStream = list.stream().map(s -> Arrays.stream(s.split(",")));
   ```

   > flatMap——接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流
   >
   > `<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)`
   >
   > 它接受一个函数mapper，对流中的每一个元素，mapper会将该元素转换为一个流Stream，然后把新生成流的每一个元素传递给下一个操作.

   ```java
   List<String> list = Arrays.asList("1,2", "3,4");
   //每次mapper操作返回一个流Stream<String> 然后将流里的每个元素添加到新流中,最终生成Stream<String>
   Stream<String> stringStream = list.stream().flatMap(s -> Arrays.stream(s.split(",")));
   ```

   > flatMap 把 Stream 中的层级结构扁平化，就是将最底层元素抽出来放到一起，最终生成的新 Stream 里面都是直接的字符串。

2. 排序(sort)

   > sorted() ——自然排序(根据流中元素实现的Comparable接口的compareTo()方法来排序的)
   >
   > sorted(Comparator com) ——定制排序(根据特定的比较器来排序)

   ```java
   List<Integer> list = Arrays.asList(1,3,4,0,9,8,5);
   Stream<Integer> sorted = list.stream().sorted();
   sorted.forEach(System.out::print);
   /*
    输出结果: 0134589
   */
   ```

   ```java
   emps.stream()
           .sorted((x, y) -> {
               if (x.getAge() == y.getAge()) {
                   return x.getName().compareTo(y.getName());
               } else {
                   return Integer.compare(x.getAge(), y.getAge());
               }
           }).forEach(System.out::println);
   /*
   	指定比较规则,按姓名排序,姓名相同的再根据年龄排序
   */
   ```

3. 筛选与切片

   > filter : 接受Lambda,从流中排除某些元素
   >
   > limit(n) : 返回流中前n个元素
   >
   > skip(n) : 跳过流中前n个元素
   >
   > distinct : 去掉流中重复元素(通过hashCode和equles方法判断是否为相同对象)

   **filter**

   ```java
   // 筛选出姓赵的员工
   Stream<Employee> resultStream = emps.stream()
           .filter(employee ->
                   employee.getName().startsWith("赵"));
   resultStream.forEach(System.out::println);
   /*
   	输出结果:
   	Employee [id=104, name=赵六, age=20, salary=7777.77]
   	Employee [id=104, name=赵六, age=19, salary=7777.77]
   	Employee [id=104, name=赵四, age=40, salary=7777.77]
   */
   ```

   **limit**

   ```java
   //获取列表前3个员工
   Stream<Employee> limit = emps.stream().limit(3);
   limit.forEach(System.out::println);
   /*
   	Employee [id=102, name=李四, age=59, salary=6666.66]
   	Employee [id=101, name=张三, age=18, salary=9999.99]
   	Employee [id=103, name=王五, age=28, salary=3333.33]
   */
   ```

   **skin**

   ```java
   //去掉前3个员工
   Stream<Employee> limit = emps.stream().skip(3);
   limit.forEach(System.out::println);
   /*
   	Employee [id=104, name=赵六, age=20, salary=7777.77]
   	Employee [id=104, name=赵六, age=19, salary=7777.77]
   	Employee [id=104, name=赵四, age=40, salary=7777.77]
   	Employee [id=105, name=田七, age=38, salary=5555.55]
   */
   ```

   **distinct**

   ```java
   List<Integer> list = Arrays.asList(1, 1, 2, 2, 3, 3, 3, 3, 4, 5, 6);
   Stream<Integer> distinct = list.stream().distinct();//去掉重复元素
   distinct.forEach(System.out::print);
   /*
   	123456
   */
   ```

#### 终止操作

1.  查找与匹配

   > allMatch——检查是否匹配所有元素
   > anyMatch——检查是否至少匹配一个元素
   > noneMatch——检查是否没有匹配的元素
   > findFirst——返回第一个元素
   > findAny——返回当前流中的任意元素
   > count——返回流中元素的总个数
   > max——返回流中最大值
   > min——返回流中最小值

   **allMatch**

   ```java
   List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);
   boolean b = list.stream().allMatch(i -> i < 10);//检查所有元素是否都小于10
   //true
   ```

   **anyMatch**

   ```java
   List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);
   boolean b = list.stream().anyMatch(i -> i < 2);//检查是否至少有一个元素小于2
   //true
   ```

   **noneMatch**

   ```java
   List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);
   boolean b = list.stream().noneMatch(i -> i < 2);//检查是否没有一个元素小于2
   //false
   ```

   **findFirst**

   ```java
   //返回list第一个元素
   Optional<Integer> any = list.stream().findFirst();
   System.out.println(any.get());
   ```

   > `Optional<T>` 类是一个是一个容器类,代表一个值存在或不存在,原来用 null 表示一个值不存在,现在`Optional<T>`可以更好的表达这个概念,并且可以避免空指针异常
   >
   > 这里findFirst()查找第一个元素有可能为空,就把结果封装成一个Optional类.

   ```java
   List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);
   long count = list.stream().count();//统计元素个数
   System.out.println(count);//6
   Optional<Integer> max = list.stream().max(Integer::compareTo);//最大值
   System.out.println(max.get());//6
   Optional<Integer> min = list.stream().min(Integer::compareTo);//最小值
   System.out.println(min.get());//1
   ```

2. 规约(reduce)

   > `reduce(T identity, BinaryOperator bo) / reduce(BinaryOperator) ——可以将流中元素按照指定的二院运算反复结合起来，得到一个值
   >
   > identity : 起始值
   >
   > BinaryOperator  : 二元运算

   ```java
   List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
   Integer sum = list.stream()
      .reduce(0, (x, y) -> x + y);
   System.out.println(sum);
   /*
   	输出结果 55
   */
   ```

   这里reduce操作中,起始值为0,第一次x为0,list中第一个元素1为y 经行(x+y)操作,然后又把(x+y)的值作为x, list中第二个元素2作为y,依次累加.最终得到一个sum值

   ```java
   Optional<Double> op = emps.stream()
      .map(Employee::getSalary)
      .reduce(Double::sum);//计算所有员工薪资总和
   System.out.println(op.get());
   ```

   > 这个地方由于没有初始值,计算结果可能为空(列表为空的情况),所以就把计算结果封装到Optional中避免空指针异常

3.  收集(collect)

   > collect——将流转换为其他形式。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法
   >
   > `<R, A> R collect(Collector<? super T, A, R> collector);`

   ```java
   //收集员工姓名到List集合
   List<String> list = emps.stream()
      .map(Employee::getName)
      .collect(Collectors.toList());
   list.forEach(System.out::print);
   // 输出: 李四张三王五赵六赵六赵六田七
   ```

   ```java
   //收集员工姓名到Set集合
   Set<String> set = emps.stream()
      .map(Employee::getName)
      .collect(Collectors.toSet());
   set.forEach(System.out::println);
   // 输出 : 李四张三王五赵六田七
   //------------------------------------
   HashSet<String> hs = emps.stream()
       .map(Employee::getName)
       .collect(Collectors.toCollection(HashSet::new));
   hs.forEach(System.out::println);
   // 输出 : 李四张三王五赵六田七
   ```

4.  分组

   > groupingBy : 根据指定的元素对流中数据进行分组

   ```java
   //按照员工姓氏分组,这里不考虑复姓
   Map<String, List<Employee>> collect = emps.stream()
           .collect(Collectors
                   .groupingBy(emp -> String.valueOf(emp.getName().charAt(0))));
   collect.forEach((k, v) -> {
       System.out.println(k + ":" + v);
   });
   ```


   > 输出结果为:
   >
   > 田:[Employee [id=105, name=田七, age=38, salary=5555.55]
   >
   > 张:[Employee [id=101, name=张三, age=18, salary=9999.99]
   >
   > 赵:[Employee [id=104, name=赵六, age=20, salary=7777.77], 
   >
   > ​     Employee [id=104, name=赵六, age=19, salary=7777.77], 
   >
   > ​     Employee [id=104, name=赵四, age=40, salary=7777.77]]
   >
   > 王:[Employee [id=103, name=王五, age=28, salary=3333.33]
   > 李:[Employee [id=102, name=李四, age=59, salary=6666.66]

5.  分区

   > partitioningBy : 按照给定条件对流中元素进行分区

   ```java
   //将员工以薪资6000.00为界限分区
   Map<Boolean, List<Employee>> collect = emps.stream()
           .collect(Collectors.partitioningBy(e -> e.getSalary() > 6000.00));
   collect.forEach((k, v) -> {
       System.out.println(k + ":" + v);
   });
   ```

   > 输出结果为:
   >
   > false:[Employee [id=103, name=王五, age=28, salary=3333.33], 
   >
   > ​	Employee [id=105, name=田七, age=38, salary=5555.55]]
   >
   > true:[Employee [id=102, name=李四, age=59, salary=6666.66], 
   >
   > ​	Employee [id=101, name=张三, age=18, salary=9999.99], 
   >
   > ​	Employee [id=104, name=赵六, age=20, salary=7777.77], 
   >
   > ​	Employee [id=104, name=赵六, age=19, salary=7777.77], 
   >
   > ​	Employee [id=104, name=赵四, age=40, salary=7777.77]]

### Optional 类

#### 介绍

> Optional 容器类：用于尽量避免空指针异常


**方法**
> Optional 容器类：用于尽量避免空指针异常
>
> Optional.of(T t) : 创建一个 Optional 实例
>
> Optional.empty() : 创建一个空的 Optional 实例
>
> Optional.ofNullable(T t):若 t 不为 null,创建 Optional 实例,否则创建空实例
>
> isPresent() : 判断是否包含值
>
> orElse(T t) :  如果调用对象包含值，返回该值，否则返回t
>
> orElseGet(Supplier s) :如果调用对象包含值，返回该值，否则返回 s 获取的值
>
> map(Function f): 如果有值对其处理，并返回处理后的Optional，否则返回 Optional.empty()
>
> flatMap(Function mapper):与 map 类似，要求返回值必须是Optional

### 小结

> Stream 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。使用Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。也可以使用 Stream API 来并行执行操作。
>
> 简而言之，Stream API 提供了一种高效且易于使用的处理数据的方式

