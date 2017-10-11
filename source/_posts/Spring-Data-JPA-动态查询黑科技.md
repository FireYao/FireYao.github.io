---
title: Spring-Data-JPA 动态查询黑科技
date: 2017-10-10 12:55:16
categories:
- spring
tags: [spring,jpa]
toc: true
reward: false
---

在开发中,用到动态查询的地方,所有的查询条件包括分页参数,都会被封装成一个查询类`XxxQuery`

比如说上一篇中的`Item`

那么`ItemQuery`就像这样

```java
@Data
public class ItemQuery {

    private Integer itemId;//id精确查询 =

    private String itemName;//name模糊查询 like

  	//价格查询
    private Integer itemPrice;// 价格小于'条件' <

}
```

那现在问题来了,如何去标识这些字段该用怎样的查询条件连接呢,还要考虑到每个查询类都可以通用.

------

<!-- more -->

可以用字段注解,来标识字段的查询连接条件

```java
//用枚举类表示查询连接条件
public enum MatchType {
    equal,        // filed = value
  	//下面四个用于Number类型的比较
    gt,   // filed > value
    ge,   // field >= value
    lt,              // field < value
    le,      // field <= value

    notEqual,            // field != value
    like,   // field like value
    notLike,    // field not like value
    // 下面四个用于可比较类型(Comparable)的比较
    greaterThan,        // field > value
    greaterThanOrEqualTo,   // field >= value
    lessThan,               // field < value
    lessThanOrEqualTo,      // field <= value
    ;
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface QueryWord {

    // 数据库中字段名,默认为空字符串,则Query类中的字段要与数据库中字段一致
    String column() default "";

    // equal, like, gt, lt...
    MatchType func() default MatchType.equal;

    // object是否可以为null
    boolean nullable() default false;

    // 字符串是否可为空
    boolean emptiable() default false;
}
```

好了,现在我们可以改造一下`ItemQuery`了

```java
@Data
public class ItemQuery {

    @QueryWord(column = "item_id", func = MatchType.equal)
    private Integer itemId;

    @QueryWord(func = MatchType.like)
    private String itemName;
  
    @QueryWord(func = MatchType.le)
    private Integer itemPrice;

}
```

现在,我们还需要去构造出查询时的动态条件,那就创建一个所有查询类的基类`BaseQuery`,我们把分页的条件字段放在基类里.

```java
/**
 * 所有查询类的基类
 */
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public abstract class BaseQuery<T> {

    // start from 0
    protected int pageIndex = 0;
    protected int pageSize = 10;


    /**
     * 将查询转换成Specification
     * @return
     */
    public abstract Specification<T> toSpec();

    //JPA分页查询类
    public Pageable toPageable() {
        return new PageRequest(pageIndex, pageSize);
    }

    //JPA分页查询类,带排序条件
    public Pageable toPageable(Sort sort) {
        return new PageRequest(pageIndex, pageSize, sort);
    }

    //动态查询and连接
    protected Specification<T> toSpecWithAnd() {
        return this.toSpecWithLogicType("and");
    }

    //动态查询or连接
    protected Specification<T> toSpecWithOr() {
        return this.toSpecWithLogicType("or");
    }

    //logicType or/and
    private Specification<T> toSpecWithLogicType(String logicType) {
        BaseQuery outerThis = this;
        return (root, criteriaQuery, cb) -> {
            Class clazz = outerThis.getClass();
			//获取查询类Query的所有字段,包括父类字段
            List<Field> fields = getAllFieldsWithRoot(clazz);
            List<Predicate> predicates = new ArrayList<>(fields.size());
            for (Field field : fields) {
              	//获取字段上的@QueryWord注解
                QueryWord qw = field.getAnnotation(QueryWord.class);
                if (qw == null)
                    continue;

                // 获取字段名
                String column = qw.column();
                //如果主注解上colume为默认值"",则以field为准
                if (column.equals(""))
                    column = field.getName();

                field.setAccessible(true);

                try {

                    // nullable
                    Object value = field.get(outerThis);
                  	//如果值为null,注解未标注nullable,跳过
                    if (value == null && !qw.nullable())
                        continue;

                    // can be empty
                    if (value != null && String.class.isAssignableFrom(value.getClass())) {
                        String s = (String) value;
                      	//如果值为"",且注解未标注emptyable,跳过
                        if (s.equals("") && !qw.emptiable())
                            continue;
                    }
					
                  	//通过注解上func属性,构建路径表达式
                    Path path = root.get(column);
                    switch (qw.func()) {
                        case equal:
                            predicates.add(cb.equal(path, value));
                            break;
                        case like:
                            predicates.add(cb.like(path, "%" + value + "%"));
                            break;
                        case gt:
                            predicates.add(cb.gt(path, (Number) value));
                            break;
                        case lt:
                            predicates.add(cb.lt(path, (Number) value));
                            break;
                        case ge:
                            predicates.add(cb.ge(path, (Number) value));
                            break;
                        case le:
                            predicates.add(cb.le(path, (Number) value));
                            break;
                        case notEqual:
                            predicates.add(cb.notEqual(path, value));
                            break;
                        case notLike:
                            predicates.add(cb.notLike(path, "%" + value + "%"));
                            break;
                        case greaterThan:
                            predicates.add(cb.greaterThan(path, (Comparable) value));
                            break;
                        case greaterThanOrEqualTo:
                            predicates.add(cb.greaterThanOrEqualTo(path, (Comparable) value));
                            break;
                        case lessThan:
                            predicates.add(cb.lessThan(path, (Comparable) value));
                            break;
                        case lessThanOrEqualTo:
                            predicates.add(cb.lessThanOrEqualTo(path, (Comparable) value));
                            break;
                    }
                } catch (Exception e) {
                    continue;
                }
            }
            Predicate p = null;
            if (logicType == null || logicType.equals("") || logicType.equals("and")) {
                p = cb.and(predicates.toArray(new Predicate[predicates.size()]));//and连接
            } else if (logicType.equals("or")) {
                p = cb.or(predicates.toArray(new Predicate[predicates.size()]));//or连接
            }
            return p;
        };
    }

    //获取类clazz的所有Field，包括其父类的Field
    private List<Field> getAllFieldsWithRoot(Class<?> clazz) {
        List<Field> fieldList = new ArrayList<>();
        Field[] dFields = clazz.getDeclaredFields();//获取本类所有字段
        if (null != dFields && dFields.length > 0)
            fieldList.addAll(Arrays.asList(dFields));

        // 若父类是Object，则直接返回当前Field列表
        Class<?> superClass = clazz.getSuperclass();
        if (superClass == Object.class) return Arrays.asList(dFields);

        // 递归查询父类的field列表
        List<Field> superFields = getAllFieldsWithRoot(superClass);

        if (null != superFields && !superFields.isEmpty()) {
            superFields.stream().
                    filter(field -> !fieldList.contains(field)).//不重复字段
                    forEach(field -> fieldList.add(field));
        }
        return fieldList;
    }
}
```

在`BaseQuery`里,就通过`toSpecWithAnd()` `toSpecWithOr()`方法动态构建出了查询条件.

那现在`ItemQuery`就要继承`BaseQuery`,并实现`toSpec()`抽象方法

```java
@Data
public class ItemQuery extends BaseQuery<Item> {

    @QueryWord(column = "item_id", func = MatchType.equal)
    private Integer itemId;

    @QueryWord(func = MatchType.like)
    private String itemName;
  
    @QueryWord(func = MatchType.le)
    private Integer itemPrice;

    @Override
    public Specification<Item> toSpec() {
        return super.toSpecWithAnd();//所有条件用and连接
    }
}
```



当然肯定还有其他不能在BaseQuery中构建的查询条件,那就在子类的toSpec()实现中添加,

比如下面的例子,`ItemQuery`条件改成这样

```java
@QueryWord(column = "item_id", func = MatchType.equal)
private Integer itemId;

@QueryWord(func = MatchType.like)
private String itemName;

//价格范围查询
private Integer itemPriceMin;
private Integer itemPriceMax;
```

那其他条件就可以在`toSpec()`添加,这样就可以很灵活的构建查询条件了

```java
@Override
public Specification<Item> toSpec() {
    Specification<Item> spec = super.toSpecWithAnd();
    return ((root, criteriaQuery, criteriaBuilder) -> {
        List<Predicate> predicatesList = new ArrayList<>();
        predicatesList.add(spec.toPredicate(root, criteriaQuery, criteriaBuilder));
        if (itemPriceMin != null) {
            predicatesList.add(
                    criteriaBuilder.and(
                            criteriaBuilder.ge(
                                    root.get(Item_.itemPrice), itemPriceMin)));
        }
        if (itemPriceMax != null) {
            predicatesList.add(
                    criteriaBuilder.and(
                            criteriaBuilder.le(
                                    root.get(Item_.itemPrice), itemPriceMax)));
        }
       return criteriaBuilder.and(predicatesList.toArray(new Predicate[predicatesList.size()]));
    });
}
```

调用:

```java
@Test
public void test1() throws Exception {
    ItemQuery itemQuery = new ItemQuery();
    itemQuery.setItemName("车");
    itemQuery.setItemPriceMax(50);
    itemQuery.setItemPriceMax(200);
    Pageable pageable = itemQuery.toPageable(new Sort(Sort.Direction.ASC, "itemId"));
    Page<Item> all = itemRepository.findAll(itemQuery.toSpec(), pageable);
}
```

现在这个`BaseQuery`和`QuertWord`就可以在各个动态查询处使用了,只需在查询字段上标注@QueryWord注解,

然后实现`BaseQuery`中的抽象方法`toSpec()`,通过`JpaSpecificationExecutor`接口中的这几个方法,就可以实现动态查询了,是不是很方便.

```java
public interface JpaSpecificationExecutor<T> {
    T findOne(Specification<T> var1);
    List<T> findAll(Specification<T> var1);
    Page<T> findAll(Specification<T> var1, Pageable var2);
    List<T> findAll(Specification<T> var1, Sort var2);
    long count(Specification<T> var1);
}
```

