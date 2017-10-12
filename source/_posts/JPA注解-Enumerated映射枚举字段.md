---
title: JPA注解@Enumerated映射枚举字段
date: 2017-10-12 10:59:34
categories:
- spring
tags: [spring,jpa,hibernate]
toc: true
reward: false
---

在javax.persistence包中有这么两个注解@Enumerated,@EnumType

```java
 */
@Target({METHOD, FIELD})
@Retention(RUNTIME)
public @interface Enumerated {

    /** (Optional) The type used in mapping an enum type. */
    EnumType value() default ORDINAL;
}
public enum EnumType {
    /** 持久枚举类型字段为整数，元素一般从0开始索引. */
    ORDINAL,

    /** 持久枚举类型为字符串. */
    STRING
}
```

当我需要持久化一个枚举类字段的时候，就可以用`@Enumerated`来标注枚举类型。来举个栗子：

<!-- more -->

数据库中有一张employee表

![](http://oqnan33k8.bkt.clouddn.com/spring/jpa/%E6%B3%A8%E8%A7%A3/employee.png)

对应的Employee实体

```java
@Entity
@Table(name = "employee", schema = "public")
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Employee {

    @Id
    @GeneratedValue
    @Column(name = "id")
    private Long id;

    @Column(name = "name")
    private String name;

    @Column(name = "sex")
    @Enumerated(EnumType.ORDINAL)//性别字段持久化为0，1
    private Sex sex;

    @Column(name = "type")
    @Enumerated(EnumType.STRING)//枚举字符串
    private Type type;
}
```

Sex枚举类：

```java
public enum Sex {

    MAIL("男"),
    FMAIL("女");

    private String value;

    private Sex(String value) {
        this.value = value;
    }
}
```

Type枚举类：

```java
public enum Type {

    PROGRAMMER("开发"),
    PM("项目经理"),
    TESTERS("测试"),
    UI("妹子"),
    ;

    private String type;

    private Type(String type) {
    }
}
```

那现在我们来看一下插入几条数据看下是什么效果。

```java
EmployeeRepostory employeeRepostory = context.getBean(EmployeeRepostory.class);

Employee fireYao = Employee.builder().name("fireYao").sex(Sex.MAIL).type(Type.PROGRAMMER).build();

Employee gakki = Employee.builder().name("gakki").sex(Sex.FMAIL).type(Type.UI).build();

Employee whoever = Employee.builder().name("whoever").sex(Sex.FMAIL).type(Type.PM).build();

employeeRepostory.save(Arrays.asList(fireYao,gakki,whoever));
```

插入数据后，数据库中：

![](http://oqnan33k8.bkt.clouddn.com/spring/jpa/%E6%B3%A8%E8%A7%A3/employee_data.png)

可以看到，sex字段被持久化为0,1这样的int字段，因为在sex字段上标注了`@Enumerated(EnumType.ORDINAL)`，那这样持久化到数据库时，就会根据枚举类中的字段，依次从0开始标记，Sex中` MAIL("男")`就为0,` FMAIL("女")`就为1,根据枚举类的字段个数依次递增。

在type字段上标注了`@Enumerated(EnumType.STRING)`,就直接根据枚举类的字段字符串来自动持久化到数据库。

如果枚举字段上不加注解，那么枚举字段就会被默认映射为 int 类型存储。

那现在我们来看查询一下的结果。

```java
List<Employee> all = employeeRepostory.findAll();
```

![](http://oqnan33k8.bkt.clouddn.com/spring/jpa/%E6%B3%A8%E8%A7%A3/employee_result.png)

可以看到，查询出来的结果中sex字段又自动还原成了Sex枚举字段。

------

