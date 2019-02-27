## JpaRepository 扩展之 QueryByExampleExecutor
### JpaRepository 介绍

从 JpaRepository 开始的子类，都是 Spring Data 项目对 JPA 实现的封装与扩展。JpaRepository 本身继承 PagingAndSortingRepository 接口，是针对 JPA 技术的接口，提供 flush()、saveAndFlush()、deleteInBatch()、deleteAllInBatch() 等方法。我们来看一下 UML 来对 JpaRespository 有个整体的认识。
![在这里插入图片描述](http://images.gitbook.cn/af50bef0-2eab-11e8-8e27-3f3d79ceb212)

- 从图中其实可以发现，JPA 的实现类最关键是：SimpleJpaRepository，我们多次提到，还有一个最关键的实现类是 QuerydslJpaRepository，会在后面继续介绍。
- 从图中还可以看出来，最关键的几个接口 QueryByExampleExecutor、JpaSpecificationExecutor。
- 从图中还可以好好体会一些接口的用意（暴露那些该暴露的操作方法，而不是一股脑的把所有的方法都暴露给使用的人，因为不是每个场景下面都会用到所有方法。作者感悟在实际工作中，当我们去设计公共方法或者架构的时候，要充分考虑清楚抽象类和接口的区别及其应用场景。）

### QueryByExampleExecutor 的使用
按示例查询（QBE）是一种用户友好的查询技术，具有简单的接口，它允许动态查询创建，并且不需要编写包含字段名称的查询。从 UML 图中，可以看出继承 JpaRepository 接口后，自动拥有了按“实例”进行查询的诸多方法，可见 Spring Data 的团队已经认为了 QBE 是 Spring JPA 的基本功能了，继承 QueryByExampleExecutor 和继承 JpaRepository 都会有这些基本方法，所以 QueryByExampleExecutor 位于 Spring Data Common 中，而 JpaRepository 位于 Spring Data JPA 中。


### QueryByExampleExecutor 详细配置
先来看一下 QueryByExampleExecutor 的源码：
```
public interface QueryByExampleExecutor<T> { 
 //根据“实例”查找一个对象。
<S extends T> S findOne(Example<S> example);
//根据“实例”查找一批对象
<S extends T> Iterable<S> findAll(Example<S> example); 
//根据“实例”查找一批对象，且排序
<S extends T> Iterable<S> findAll(Example<S> example, Sort sort);
 //根据“实例”查找一批对象，且排序和分页 
<S extends T> Page<S> findAll(Example<S> example, Pageable pageable);
//根据“实例”查找，返回符合条件的对象个数
<S extends T> long count(Example<S> example); 
//根据“实例”判断是否有符合条件的对象
<S extends T> boolean exists(Example<S> example); 
}
```

从源码上可以看出，只要了解 Example 基本上就可以掌握它的用法和 API 了。
```
public class Example<T> {
    @NonNull
    private final T probe;
    @NonNull
    private final ExampleMatcher matcher;
public static <T> Example<T> of(T probe) {
    return new Example(probe, ExampleMatcher.matching());
}
public static <T> Example<T> of(T probe, ExampleMatcher matcher) {
    return new Example(probe, matcher);
}
......
}
```

我们从源码中可以看出 Example 主要包含三部分内容。

- Probe：这是具有填充字段的域对象的实际实体类，即查询条件的封装类（又可以理解为：查询条件参数），必填。
- ExampleMatcher：ExampleMatcher 有关于如何匹配特定字段的匹配规则，它可以重复使用在多个示例，必填；如果不填，用默认的（又可以理解为参数的匹配规则）。
- Example：Example 由 Probe 探针和 ExampleMatcher 组成，它用于创建查询，即组合查询参数和参数的匹配规则。


### QueryByExampleExecutor 的使用案例
```
//创建查询条件数据对象
Customer customer = new Customer();
customer.setName("Jack");
customer.setAddress("上海");
//创建匹配器，即如何使用查询条件
ExampleMatcher matcher = ExampleMatcher.matching() //构建对象
        .withMatcher("name", GenericPropertyMatchers.startsWith()) //姓名采用“开始匹配”的方式查询
        .withIgnorePaths("focus");  //忽略属性：是否关注。因为是基本类型，需要忽略掉
//创建实例
Example<Customer> ex = Example.of(customer, matcher); 
//查询
List<Customer> ls = dao.findAll(ex);
//输出结果
for (Customer bo:ls)
{
    System.out.println(bo.getName());
}

```
上面例子中，是这样创建“实例”的：Example<Customer> ex = Example.of(customer, matcher);可以看到，**Example 对象由 customer 和 matcher 共同创建**，为讲解方便，再来结合案例先来明确一些定义。
- Probe：实体对象，在持久化框架中与 Table 对应的域对象，一个对象代表数据库表中的一条记录，如上例中 Customer 对象。在构建查询条件时，一个实体对象代表的是查询条件中的“数值”部分，如要查询姓“Jack”的客户，实体对象只能存储条件值“Jack”。
- ExampleMatcher：匹配器，它是匹配“实体对象”的，表示了如何使用“实体对象”中的“值”进行查询，它代表的是“查询方式”，解释了如何去查的问题。例如，要查询姓“刘”的客户，即姓名以“刘”开头的客户，该对象就表示了“以某某开头的”这个查询方式，如上例中 withMatcher("name", GenericPropertyMatchers.startsWith())。
- Example：实例对象，代表的是完整的查询条件。由实体对象（查询条件值）和匹配器（查询方式）共同创建。
再来理解“实例查询”，顾名思义，就是通过一个例子来查询，要查询的是 Customer 对象，查询条件也是一个 Customer 对象，通过一个现有的客户对象作为例子，查询和这个例子相匹配的对象。

### QueryByExampleExecutor 的特点及约束
- 支持动态查询：即支持查询条件个数不固定的情况，如客户列表中有多个过滤条件，用户使用时在“地址”查询框中输入了值，就需要按地址进行过滤，如果没有输入值，就忽略这个过滤条件。对应的实现是，在构建查询条件 Customer 对象时，将 address 属性值置具体的条件值或置为 null。
- 不支持过滤条件分组：即不支持过滤条件用 or（或）来连接，所有的过滤查件，都是简单一层的用 and（并且）连接，如 firstname = ?0 or (firstname = ?1 and lastname = ?2)。
- 仅支持字符串的开始/包含/结束/正则表达式匹配和其他属性类型的精确匹配。查询时，对一个要进行匹配的属性（如：姓名 name），只能传入一个过滤条件值，如以 Customer 为例，要查询姓“刘”的客户，“刘”这个条件值就存储在表示条件对象的 Customer 对象的 name 属性中，针对于“姓名”的过滤也只有这么一个存储过滤值的位置，没办法同时传入两个过滤值。正是由于这个限制，有些查询是没办法支持的，例如要查询某个时间段内添加的客户，对应的属性是 addTime，需要传入“开始时间”和“结束时间”两个条件值，而这种查询方式没有存两个值的位置，所以就没办法完成这样的查询。

### ExampleMatcher 源码解读
（1）源码解读
```
public class ExampleMatcher {
NullHandler nullHandler; 
StringMatcher defaultStringMatcher; //默认
boolean defaultIgnoreCase; //默认大小写忽略方式
PropertySpecifiers propertySpecifiers; //各属性特定查询方式
Set<String> ignoredPaths; //忽略属性列表
//Null值处理方式，通过构造方法，我们发现默认忽略
   NullHandler nullHandler;
   //字符串匹配方式,通过构造方法可以看出默认是DEFAULT（默认，效果同EXACT）,EXACT（相等）
   StringMatcher defaultStringMatcher;
   //各属性特定查询方式，默认无特殊指定的。
   PropertySpecifiers propertySpecifiers;
   //忽略属性列表，默认无。
   Set<String> ignoredPaths;
   //大小写忽略方式,默认不忽略。
   boolean defaultIgnoreCase;
   @Wither(AccessLevel.PRIVATE) MatchMode mode;
//通用、内部、默认构造方法。
   private ExampleMatcher() {
      this(NullHandler.IGNORE, StringMatcher.DEFAULT, new PropertySpecifiers(), Collections.<String>emptySet(), false,
            MatchMode.ALL);
   }
   //Example的默认匹配方式
   public static ExampleMatcher matching() {
      return matchingAll();
   }
public static ExampleMatcher matchingAll() {
   return new ExampleMatcher().withMode(MatchMode.ALL);
}
......
}

```

（2）关键属性分析

1.nullHandler：Null 值处理方式，枚举类型，有两个可选值，INCLUDE（包括）、IGNORE（忽略）。
标识作为条件的实体对象中，一个属性值（条件值）为 Null 时，是否参与过滤；
当该选项值是 INCLUDE 时，表示仍参与过滤，会匹配数据库表中该字段值是 Null 的记录；
若为 IGNORE 值，表示不参与过滤。
2.defaultStringMatcher：默认字符串匹配方式，枚举类型，有 6 个可选值，**DEFAULT（默认，效果同 EXACT）、EXACT（相等）、STARTING（开始匹配）、ENDING（结束匹配）、CONTAINING（包含，模糊匹配）、REGEX（正则表达式）**。
该配置对所有字符串属性过滤有效，除非该属性在 propertySpecifiers 中单独定义自己的匹配方式。
3.defaultIgnoreCase：默认大小写忽略方式，布尔型，当值为 false 时，即不忽略，大小不相等。
该配置对所有字符串属性过滤有效，除非该属性在 propertySpecifiers 中单独定义自己的忽略大小写方式。
4.propertySpecifiers：各属性特定查询方式，描述了各个属性单独定义的查询方式，每个查询方式中包含4个元素：属性名、字符串匹配方式、大小写忽略方式、属性转换器。
如果属性未单独定义查询方式，或单独查询方式中，某个元素未定义（如字符串匹配方式），则采用 ExampleMatcher 中定义的默认值，即上面介绍的 defaultStringMatcher 和 defaultIgnoreCase 的值。
5.ignoredPaths：忽略属性列表，忽略的属性不参与查询过滤。


（3）字符串匹配举例

**字符串匹配方式	对应 JPQL 的写法**
Default& 不忽略大小写	firstname=?1
Exact& 忽略大小写	LOWER(firstname) = LOWER(?1)
Staring& 忽略大小写	LOWER(firstname) like LOWER(?0)+'%'
Ending& 不忽略大小写	firstname like '%'+?1
Containing 不忽略大小写	firstname like '%'+?1+'%'


### QueryByExampleExecutor 使用场景 & 实际的使用
#### 使用场景
使用一组静态或动态约束来查询数据存储、频繁重构域对象，而不用担心破坏现有查询、简单的查询的使用场景，有时候还是挺方便的。

#### 实际使用中我们需要考虑的因素
查询条件的表示，有两部分，一是条件值，二是查询方式。条件值用实体对象（如 Customer 对象）来存储，相对简单，当页面传入过滤条件值时，存入相对应的属性中，没入传入时，属性保持默认值。查询方式是用匹配器 ExampleMatcher 来表示，情况相对复杂些，需要考虑的因素有以下几个：

（1）Null 值的处理

当某个条件值为 Null时，是应当忽略这个过滤条件呢，还是应当去匹配数据库表中该字段值是 Null 的记录？

Null 值处理方式：**默认值是 IGNORE（忽略），即当条件值为 Null 时，则忽略此过滤条件，一般业务也是采用这种方式就可满足**。当需要查询数据库表中属性为 Null 的记录时，**可将值设为 INCLUDE**，这时，对于不需要参与查询的属性，都必须添加到忽略列表（ignoredPaths）中，否则会出现查不到数据的情况。

（2）基本类型的处理

如客户 Customer 对象中的年龄 age 是 int 型的，当页面不传入条件值时，它默认是0，是有值的，那是否参与查询呢？

关于基本数据类型处理方式：**实体对象中，避免使用基本数据类型，采用包装器类型**。如果已经采用了基本类型，而这个属性查询时不需要进行过滤，则把它添加到忽略列表（ignoredPaths）中。

（3）忽略某些属性值

一个实体对象，有许多个属性，是否每个属性都参与过滤？是否可以忽略某些属性？

ignoredPaths：虽然某些字段里面有值或者设置了其他匹配规则，**只要放在 ignoredPaths 中，就会忽略此字段的，不作为过滤条件**。

（4）不同的过滤方式

同样是作为 String 值，可能“姓名”希望精确匹配，“地址”希望模糊匹配，如何做到？

默认配置和特殊配置混合使用：默认创建匹配器时，字符串采用的是精确匹配、不忽略大小写，可以通过操作方法改变这种默认匹配，以满足大多数查询条件的需要，如将“字符串匹配方式”改为 CONTAINING（包含，模糊匹配），这是比较常用的情况。对于个别属性需要特定的查询方式，可以通过配置“属性特定查询方式”来满足要求，设置 propertySpecifiers 的值即可。

（5）大小写匹配

字符串匹配时，有时可能希望忽略大小写，有时则不忽略，如何做到？

defaultIgnoreCase：忽略大小的生效与否，是依赖于数据库的。例如 MySQL 数据库中，默认创建表结构时，字段是已经忽略大小写的，所以这个配置与否，都是忽略的。如果业务需要严格区分大小写，可以改变数据库表结构属性来实现。


### 实际使用案例说明
（1）无匹配器的情况

要求：查询地址是“河南省郑州市”，且重点关注的客户。
说明：使用默认匹配器就可以满足查询条件，则不需要创建匹配器。
```
//创建查询条件数据对象
Customer customer= new Customer();
customer.setAddress("河南省郑州市");
customer.setFocus(true); 
//创建实例
Example<Customer> ex = Example.of(customer);
//查询
List<Customer> ls = dao.findAll(ex);

```

（2）多种条件组合

要求：根据姓名、地址、备注进行模糊查询，忽略大小写，地址要求开始匹配。
说明：这是通用情况，主要演示改变默认字符串匹配方式、改变默认大小写忽略方式、属性特定查询方式配置、忽略属性列表配置。
```
 //创建查询条件数据对象
Customer customer = new Customer();
customer.setName("zhang");
customer.setAddress("河南省");
customer.setRemark("BB");
//虽然有值，但是不参与过滤条件
customer.setFocus(true);
//创建匹配器，即如何使用查询条件
ExampleMatcher matcher = ExampleMatcher.matching()//构建对象
					.withStringMatcher(StringMatcher.CONTAINING) //改变默认字符串匹配方式：模糊查询 CONTAINING
					.withIgnoreCase(true)
					.withMatcher("address",GenericPropertyMatchers.startsWith()) ////地址采用“开始匹配”的方式查询
					.withIgnorePaths("focus"); //忽略属性：是否关注。因为是基本类型，需要忽略掉
//创建实例
Example<Customer> ex = Example.of(customer,matcher);
//查询
List<Customer> ls = dao.findAll(ex);
```

（3）多级查询

要求：查询所有潜在客户。
说明：主要演示多层级属性查询。
```
//创建查询条件数据对象
CustomerType type = new CustomerType();
type.setCode("01"); //编号01代表潜在客户
Customer customer = new Customer();
customer.setCustomerType(type);        
//创建匹配器，即如何使用查询条件
ExampleMatcher matcher = ExampleMatcher.matching()//构建对象
						.withIgnorePaths("focus");  //忽略属性：是否关注。因为是基本类型，需要忽略掉    (如果改成包装类型,就可以免去考虑了)  
//创建实例
Example<Customer> ex = Example.of(customer, matcher); 
//查询
List<Customer> ls = dao.findAll(ex);						
```

（4）查询 Null 值

要求：地址是 Null 的客户。
说明：主要演示改变“Null 值处理方式”。
```
Customer customer = new Customer();
customer.setAddress(null);
//创建匹配器，即如何使用查询条件
ExampleMatcher matcher = ExampleMatcher.matching()//构建对象
						//改变“Null值处理方式”：包括。
						.withIncludeNullValues() 
						//忽略其他属性
						.withIgnorePaths("id", "name", "sex", "age", "focus", "addTime", "remark", "customerType"); 
//创建实例
Example<Customer> ex = Example.of(customer, matcher);
//查询
List<Customer> ls = dao.findAll(ex);   
```






















