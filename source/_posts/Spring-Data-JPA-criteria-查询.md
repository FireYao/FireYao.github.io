---
title: Spring-Data-JPA criteria 查询
date: 2017-10-10 12:54:47
categories:
- spring
tags: [spring,jpa]
toc: true
reward: false
---

Spring Data JPA虽然大大的简化了持久层的开发,但是在实际开发中,很多地方都需要高级动态查询

**Criteria API**

> Criteria 查询是以元模型的概念为基础的，元模型是为具体持久化单元的受管实体定义的，这些实体可以是实体类，嵌入类或者映射的父类。
>
> CriteriaQuery接口：代表一个specific的顶层查询对象，它包含着查询的各个部分，比如：select 、from、where、group by、order by等注意：CriteriaQuery对象只对实体类型或嵌入式类型的Criteria查询起作用
>
>  Root接口：代表Criteria查询的根对象，Criteria查询的查询根定义了实体类型，能为将来导航获得想要的结果，它与SQL查询中的FROM子句类似
>
> ​    1：Root实例是类型化的，且定义了查询的FROM子句中能够出现的类型。
>
> ​    2：查询根实例能通过传入一个实体类型给 AbstractQuery.from方法获得。
>
> ​    3：Criteria查询，可以有多个查询根。
>
> ​    4：AbstractQuery是CriteriaQuery 接口的父类，它提供得到查询根的方法。CriteriaBuilder接口：用来构建CritiaQuery的构建器对象Predicate：一个简单或复杂的谓词类型，其实就相当于条件或者是条件组合

> 如果编译器能够对查询执行语法正确性检查，那么对于 Java 对象而言该查询就是类型安全的。Java™Persistence API (JPA) 的 2.0 版本引入了 Criteria API，这个 API 首次将类型安全查询引入到 Java 应用程序中，并为在运行时动态地构造查询提供一种机制。

**JPA元模型**

> 在JPA中,标准查询是以元模型的概念为基础的.元模型是为具体持久化单元的受管实体定义的.这些实体可以是实体类,嵌入类或者映射的父类.提供受管实体元信息的类就是元模型类.
>
> 使用元模型类最大的优势是凭借其实例化可以在编译时访问实体的持久属性.该特性使得criteria 查询更加类型安全.

<!-- more -->

如下,`Item`实体类对应的元模型`Item_`

```java
@Generated(value = "org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor")
@StaticMetamodel(Item.class)
public abstract class Item_ {
   public static volatile SingularAttribute<Item, Integer> itemId;
   public static volatile SingularAttribute<Item, String> itemName;
   public static volatile SingularAttribute<Item, Integer> itemStock;
   public static volatile SingularAttribute<Item, Integer> itemPrice;
}
```

这样的元模型不用手动创建,在Maven中添加插件,编译之后@Entity注解的类就会自动生成对应的元模型

```xml
<!--hibernate JPA 自动生成元模型-->
<!-- 相关依赖 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-jpamodelgen</artifactId>
            <version>5.2.10.Final</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.1.0.Final</version>
        </dependency>
```

```xml
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
       <configuration>
           <source>1.8</source>
             <target>1.8</target>
              <compilerArguments>
                 <processor>org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor</processor>
              </compilerArguments>
     	</configuration>
 </plugin>
```

**使用criteria 查询简单Demo**

```java
@Service
public class ItemServiceImpl implements ItemService {

    @Resource
    private EntityManager entityManager;

    @Override
    public List<Item> findByConditions(String name, Integer price, Integer stock) {
      	//创建CriteriaBuilder安全查询工厂
        //CriteriaBuilder是一个工厂对象,安全查询的开始.用于构建JPA安全查询.
        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        //创建CriteriaQuery安全查询主语句
        //CriteriaQuery对象必须在实体类型或嵌入式类型上的Criteria 查询上起作用。
        CriteriaQuery<Item> query = criteriaBuilder.createQuery(Item.class);
        //Root 定义查询的From子句中能出现的类型
        Root<Item> itemRoot = query.from(Item.class);
      	//Predicate 过滤条件 构建where字句可能的各种条件
      	//这里用List存放多种查询条件,实现动态查询
        List<Predicate> predicatesList = new ArrayList<>();
      	//name模糊查询 ,like语句
		if (name != null) {
            predicatesList.add(
                    criteriaBuilder.and(
                            criteriaBuilder.like(
                                    itemRoot.get(Item_.itemName), "%" + name + "%")));
        }
     	// itemPrice 小于等于 <= 语句
        if (price != null) {
            predicatesList.add(
                    criteriaBuilder.and(
                            criteriaBuilder.le(
                                    itemRoot.get(Item_.itemPrice), price)));
        }
        //itemStock 大于等于 >= 语句
        if (stock != null) {
            predicatesList.add(
                    criteriaBuilder.and(
                            criteriaBuilder.ge(
                                    itemRoot.get(Item_.itemStock), stock)));
        }
      	//where()拼接查询条件
        query.where(predicatesList.toArray(new Predicate[predicatesList.size()]));
        TypedQuery<Item> typedQuery = entityManager.createQuery(query);
        List<Item> resultList = typedQuery.getResultList();
        return resultList;
    }
}
```

**criteriaBuilder中各方法对应的语句**

> equle : filed = value
>
> gt / greaterThan : filed > value
>
> lt / lessThan : filed < value
>
> ge / greaterThanOrEqualTo : filed >= value
>
> le / lessThanOrEqualTo: filed <= value
>
> notEqule : filed != value
>
> like : filed like value
>
> notLike : filed not like value

如果每个动态查询的地方都这么写,那就感觉太麻烦了.

**那实际上,在使用Spring Data JPA的时候，只要我们的Repo层接口继承JpaSpecificationExecutor接口就可以使用Specification进行动态查询了，我们先看下JpaSpecificationExecutor接口：**

```java
public interface JpaSpecificationExecutor<T> {
    T findOne(Specification<T> var1);

    List<T> findAll(Specification<T> var1);

    Page<T> findAll(Specification<T> var1, Pageable var2);

    List<T> findAll(Specification<T> var1, Sort var2);

    long count(Specification<T> var1);
}
```

在这里有个很重要的接口`Specification`

```java
public interface Specification<T> {
    Predicate toPredicate(Root<T> var1, CriteriaQuery<?> var2, CriteriaBuilder var3);
}
```

这个接口只有一个方法,返回动态查询的数据结构,用于构造各种动态查询的SQL

**Specification接口示例**

```java
public Page<Item> findByConditions(String name, Integer price, Integer stock, Pageable page) {
     Page<Item> page = itemRepository.findAll((root, criteriaQuery, criteriaBuilder) -> {
            List<Predicate> predicatesList = new ArrayList<>();
            //name模糊查询 ,like语句
            if (name != null) {
                predicatesList.add(
                        criteriaBuilder.and(
                                criteriaBuilder.like(
                                        root.get(Item_.itemName), "%" + name + "%")));
            }
            // itemPrice 小于等于 <= 语句
            if (price != null) {
                predicatesList.add(
                        criteriaBuilder.and(
                                criteriaBuilder.le(
                                        root.get(Item_.itemPrice), price)));
            }
            //itemStock 大于等于 >= 语句
            if (stock != null) {
                predicatesList.add(
                        criteriaBuilder.and(
                                criteriaBuilder.ge(
                                        root.get(Item_.itemStock), stock)));
            }
            return criteriaBuilder.and(
                    predicatesList.toArray(new Predicate[predicatesList.size()]));
        }, page);
    return page;
}
```

> 在这里因为`findAll(Specification<T> var1, Pageable var2)`方法中参数 `Specification<T>` 是一个匿名内部类
>
> 那这里就可以直接用lambda表达式直接简化代码.
>
> 这样写,就比用CriteriaBuilder安全查询工厂简单多了.

调用:

```java
Page<Item> itemPageList = findByConditions("车", 300, null, new PageRequest(1, 10));
```

利用JPA的`Specification<T>`接口和元模型就实现动态查询了.

那其实这样每一个需要动态查询的地方都需要写一个这样类似的`findByConditions`方法,感觉也很麻烦了.当然是越简化越好.

下一篇将会讲一个JPA`Specification`更方便的使用.