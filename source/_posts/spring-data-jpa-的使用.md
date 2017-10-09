---
title: Spring Data JPA 的使用
date: 2017-10-09 16:38:04
categories:
- spring
tags: [spring,jpa]
toc: true
reward: false
---

上一篇Spring JavaConfig中配置数据源使用了JPA,这里就介绍一下Spring data jpa的常用方法.

### spring data jpa介绍

#### 什么是JPA

> JPA(Java Persistence API)是Sun官方提出的Java持久化规范。它为Java开发人员提供了一种对象/关联映射工具来管理Java应用中的关系数据。

Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架，可使开发者用极简的代码即可实现对数据的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！学习并使用 Spring Data JPA 可以极大提高开发效率！

> spring data jpa让我们解脱了DAO层的操作，基本上所有CRUD都可以依赖于它来实现

**简单查询**

> 基本查询也分为两种，一种是spring data默认已经实现，一种是根据查询的方法来自动解析成SQL。
>
> spring data jpa 默认预先生成了一些基本的CURD的方法，例如：增、删、改等等

```java
public interface ItemRepository extends JpaRepository<Item, Integer>, JpaSpecificationExecutor<Item> {
//空的，可以什么都不用写
}
```

<!-- more -->

```java
@Test
public void test1() throws Exception {
    Item item = new Item();
    itemRepository.save(item);
    List<Item> itemList = itemRepository.findAll();
    Item one = itemRepository.findOne(1);
    itemRepository.delete(item);
    long count = itemRepository.count();
}
```

**自定义简单查询**

```java
Item findByItemName(String itemName);

List<Item> findByItemNameLike(String itemName);

Long deleteByItemId(Integer id);

List<Item> findByItemNameLikeOrderByItemNameDesc(String itemName);
```

**具体的关键字，使用方法和生产成SQL如下表所示**

| Keyword           | Sample                                  | JPQL snippet                             |
| ----------------- | --------------------------------------- | ---------------------------------------- |
| And               | findByLastnameAndFirstname              | … where x.lastname = ?1 and x.firstname = ?2 |
| Or                | findByLastnameOrFirstname               | … where x.lastname = ?1 or x.firstname = ?2 |
| Is,Equals         | findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                 |
| Between           | findByStartDateBetween                  | … where x.startDate between ?1 and ?2    |
| LessThan          | findByAgeLessThan                       | … where x.age < ?1                       |
| LessThanEqual     | findByAgeLessThanEqual                  | … where x.age ⇐ ?1                       |
| GreaterThan       | findByAgeGreaterThan                    | … where x.age > ?1                       |
| GreaterThanEqual  | findByAgeGreaterThanEqual               | … where x.age >= ?1                      |
| After             | findByStartDateAfter                    | … where x.startDate > ?1                 |
| Before            | findByStartDateBefore                   | … where x.startDate < ?1                 |
| IsNull            | findByAgeIsNull                         | … where x.age is null                    |
| IsNotNull,NotNull | findByAge(Is)NotNull                    | … where x.age not null                   |
| Like              | findByFirstnameLike                     | … where x.firstname like ?1              |
| NotLike           | findByFirstnameNotLike                  | … where x.firstname not like ?1          |
| StartingWith      | findByFirstnameStartingWith             | … where x.firstname like ?1 (parameter bound with appended %) |
| EndingWith        | findByFirstnameEndingWith               | … where x.firstname like ?1 (parameter bound with prepended %) |
| Containing        | findByFirstnameContaining               | … where x.firstname like ?1 (parameter bound wrapped in %) |
| OrderBy           | findByAgeOrderByLastnameDesc            | … where x.age = ?1 order by x.lastname desc |
| Not               | findByLastnameNot                       | … where x.lastname <> ?1                 |
| In                | findByAgeIn(Collection ages)            | … where x.age in ?1                      |
| NotIn             | findByAgeNotIn(Collection age)          | … where x.age not in ?1                  |
| TRUE              | findByActiveTrue()                      | … where x.active = true                  |
| FALSE             | findByActiveFalse()                     | … where x.active = false                 |
| IgnoreCase        | findByFirstnameIgnoreCase               | … where UPPER(x.firstame) = UPPER(?1)    |

**分页查询**

```java
Page<Item> findALL(Pageable pageable);
```

```java
@Test
public void test1() throws Exception {
    int page=1,size=10;
    Sort sort = new Sort(Sort.Direction.DESC, "id");//根据id降序排序
    Pageable pageable = new PageRequest(page, size, sort);
    Page<Item> pageResult = itemRepository.findALL(pageable);
    List<Item> itemList = pageResult.getContent();
}
```

**自定义SQL查询**

> 在SQL的查询方法上面使用`@Query`注解，如涉及到删除和修改在需要加上`@Modifying`.也可以根据需要添加 `@Transactional` 对事物的支持

```java
//自定分页查询 一条查询数据,一条查询数据量
@Query(value = "select i from Item i",
        countQuery = "select count(i.itemId) from Item i")
Page<Item> findall(Pageable pageable);

//nativeQuery = true 本地查询  就是使用原生SQL查询
@Query(value = "select * from item  where item_id = ?1", nativeQuery = true)
Item findAllItemById(int id);

@Transactional
@Modifying
@Query("delete from Item i where i.itemId = :itemId")
void deleteInBulkByItemId(@Param(value = "itemId") Integer itemId);

//#{#entityName}就是指定的@Entity,这里就是Item
 @Query("select i from #{#entityName} i where i.itemId = ?1")
 Item findById(Integer id);
```

**命名查询**

> 在实体类上使用@NameQueries注解
>
> 在自己实现的DAO的Repository接口里面定义一个同名的方法
>
> 然后就可以使用了，Spring会先找是否有同名的NamedQuery，如果有，那么就不会按照接口定义的方法来解析。

```java
//命名查询
@NamedQueries({
        @NamedQuery(name = "Item.findItemByitemPrice",
                query = "select i from Item i where i.itemPrice between ?1 and ?2"),
        @NamedQuery(name = "Item.findItemByitemStock",
                query = "select i from Item i where i.itemStock between ?1 and ?2"),
})
@Entity
@Data
public class Item implements Serializable {
    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private int itemId;
    private String itemName;
    private Integer itemPrice;
    private Integer itemStock;
 }
```

```java
/**
 * 这里是在domain实体类里@NamedQuery写对应的HQL
 * @NamedQuery(name = "Item.findItemByitemPrice",
               baseQuery = "select i from Item i where i.itemPrice between ?1 and ?2"),
 * @param price1
 * @param price2
 * @return
 */
List<Item> findItemByitemPrice(Integer price1, Integer price2);
List<Item> findItemByitemStock(Integer stock1, Integer stock2);
```

那么spring data jpa是怎么通过这些规范来进行组装成查询语句呢?

#### Spring Data JPA框架在进行方法名解析时，会先把方法名多余的前缀截取掉，比如 find、findBy、read、readBy、get、getBy，然后对剩下部分进行解析。

##### **假如创建如下的查询：`findByUserDepUuid()`，框架在解析该方法时，首先剔除 findBy，然后对剩下的属性进行解析**

1. 先判断 userDepUuid （根据 POJO 规范，首字母变为小写）是否为查询实体的一个属性，如果是，则表示根据该属性进行查询；如果没有该属性，继续第二步；
2. 从右往左截取第一个大写字母开头的字符串此处为Uuid），然后检查剩下的字符串是否为查询实体的一个属性，如果是，则表示根据该属性进行查询；如果没有该属性，则重复第二步，继续从右往左截取；最后假设user为查询实体的一个属性；
3. 接着处理剩下部分（DepUuid），先判断 user 所对应的类型是否有depUuid属性，如果有，则表示该方法最终是根据 ` Doc.user.depUuid` 的取值进行查询；否则继续按照步骤 2 的规则从右往左截取，最终表示根据 `Doc.user.dep.uuid` 的值进行查询。
4. 可能会存在一种特殊情况，比如 Doc包含一个 user 的属性，也有一个 userDep 属性，此时会存在混淆。可以明确在属性之间加上 "_" 以显式表达意图，比如 `findByUser_DepUuid()` 或者 `findByUserDep_uuid()`

