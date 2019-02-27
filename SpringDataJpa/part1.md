### JPA 的介绍以及哪些开源实现
JPA（Java Persistence API）中文名 Java 持久层 API，是 JDK 5.0 注解或 XML 描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。

Sun 引入新的 JPA ORM 规范出于两个原因：其一，简化现有 Java EE 和 Java SE 应用开发工作；其二，Sun 希望整合 ORM 技术，实现天下归一。

### JPA 包括以下三方面的内容
1.一套 API 标准，在 javax.persistence 的包下面，用来操作实体对象，执行 CRUD 操作，框架在后台替代我们完成所有的事情，开发者从繁琐的 JDBC 和 SQL 代码中解脱出来。
面向对象的查询语言：Java Persistence Query Language（JPQL），这是持久化操作中很重要的一个方面，通过面向对象而非面向数据库的查询语言查询数据，避免程序的 SQL 语句紧密耦合。
2.ORM（Object/Relational Metadata）元数据的映射，JPA 支持 XML 和 JDK 5.0 注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中。

### Spring Data 介绍
Spring Data 项目是从 2010 年开发发展起来的，从创立之初 Spring Data 就想提供一个大家熟悉的、一致的、基于 Spring 的数据访问编程模型，同时仍然保留底层数据存储的特殊特性。它可以轻松地让开发者使用数据访问技术包括：关系数据库、非关系数据库（NoSQL）和基于云的数据服务。

Spring Data Common 是 Spring Data 所有模块的公用部分，该项目提供跨 Spring 数据项目的共享基础设施，它包含了技术中立的库接口以及一个坚持 Java 类的元数据模型。

Spring Data 不仅对传统的数据库访问技术：JDBC、Hibernate、JDO、TopLick、JPA、MyBatis 做了很好的支持和扩展、抽象、提供方便的 API，还对 NoSQL 等非关系数据做了很好的支持：MongoDB 、Redis、Apache Solr 等。


### Spring Data JPA 的主要类及结构图
#### 我们需要掌握和使用到的类
**七个大 Repository 接口：**
	Repository（org.springframework.data.repository）；
	CrudRepository（org.springframework.data.repository）；
	PagingAndSortingRepository（org.springframework.data.repository）；
	JpaRepository（org.springframework.data.jpa.repository）；
	QueryByExampleExecutor（org.springframework.data.repository.query）；
	JpaSpecificationExecutor（org.springframework.data.jpa.repository）；
	QueryDslPredicateExecutor（org.springframework.data.querydsl）。
**两大 Repository 实现类：**
	SimpleJpaRepository（org.springframework.data.jpa.repository.support）；
	QueryDslJpaRepository（org.springframework.data.jpa.repository.support）。
	
	
	
	
## 定义查询方法（Defining Query Methods）
### 定义查询方法的配置方法

```
在这里插入代码片
```
```
示例：选择性地暴露CRUD方法
@NoRepositoryBeaninterface
MyBaseRepository<T, ID extends Serializable> extends Repository<T, ID> {
    T findOne(ID id); 
    T save(T entity);
}
interface UserRepository extends MyBaseRepository<User, Long> {
     User findByEmailAddress(EmailAddress emailAddress);
}
```

在此实例中，您为所有域存储库定义了一个公共基础接口，并将其暴露出来，findOne(…) 和 save(…) 这些方法将由 Spring Data 路由到你提供的 MyBaseRepository 的基本 Repository 实现中。在 JPA 的默认情况下，SimpleJpaRepository 作为上面两个接口的实现类，所以 UserRepository 现在将能够保存用户，并通过 ID 查找单个，以及触发查询以 Users 通过其电子邮件地址查找。
综上所述，得出以下两单：
	1.MyRepository Extends Repository 接口就可以实现 Defining Query Methods 的功能。
	2.继承其他 Repository 的子接口，或者自定义子接口，可以选择性的暴漏 SimpleJpaRepository 里面已经实现的基础公用方法。

### 方法的查询策略设置
通过下面的命令来配置方法的查询策略：
```
@EnableJpaRepositories(queryLookupStrategy= QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)
```

其中，QueryLookupStrategy.Key 的值一共就三个：

1.Create：直接根据方法名进行创建，规则是根据方法名称的构造进行尝试，一般的方法是从方法名中删除给定的一组已知前缀，并解析该方法的其余部分。如果方法名不符合规则，启动的时候会报异常。
2.USE_DECLARED_QUERY：声明方式创建，即本书说的注解的方式。启动的时候会尝试找到一个声明的查询，如果没有找到将抛出一个异常，查询可以由某处注释或其他方法声明。
3.CREATE_IF_NOT_FOUND：这个是默认的，以上两种方式的结合版。先用声明方式进行查找，如果没有找到与方法相匹配的查询，那用 Create 的方法名创建规则创建一个查询。

除非有特殊需求，一般直接用默认的，不用管。


### 查询方法的创建
内部基础架构中有个根据方法名的查询生成器机制，对于在存储库的实体上构建约束查询很有用，该机制方法的前缀 find…By、read…By、query…By、count…By 和 get…By 从所述方法和开始分析它的其余部分（实体里面的字段）。

感兴趣的读者可以到类 org.springframework.data.repository.query.parser.PartTree 查看相关源码的逻辑和处理方法，关键源码如下：

### 方法的查询策略的属性表达式（Property Expressions）
属性表达式只能引用托管（泛化）实体的直接属性，如前一个示例所示。在查询创建时，已经确保解析的属性是托管实体的属性，但是，还可以通过遍历嵌套属性定义约束。假设一个 Person 实体对象里面有一个 Address 的属性里面包含一个 ZipCode 属性。

在这种情况下，方法名为：
```
List<Person> findByAddressZipCode(String zipCode);
```
创建及其查找的过程是：解析算法首先将整个 part（AddressZipCode）解释为属性，并使用该名称（uncapitalized）检查域类的属性，如果算法成功，则使用该属性，如果不是，则算法拆分了从右侧的驼峰部分的信号源到头部和尾部，并试图找出相应的属性。在我们的例子中，AddressZip 和 Code 如果算法找到一个具有该头部的属性，那么它需要尾部，并从那里继续构建树，然后按照刚刚描述的方式将尾部分割，如果第一个分割不匹配，则算法将分割点移动到左（Address，ZipCode），然后继续。
虽然这在大多数情况下应该起作用，但算法可能会选择错误的属性。假设 Person 该类也有一个 addressZip 属性，该算法将在第一个分割轮中匹配，并且基本上选择错误的属性，最后失败（因为该类型 addressZip 可能没有 code 属性）。
要解决这个歧义，可以在方法名称中使用手动定义遍历点，所以我们的方法名称最终会如此：
```
List<Person> findByAddress_ZipCode(ZipCode zipCode);
```

### 查询结果的处理

#### 参数选择（Sort/Pageable）分页和排序
##### 特定类型的参数，Pageable 并动态 Sort 地将分页和排序应用于查询
案例：在查询方法中使用 Pageable、Slice 和 Sort。
```
Page<User> findByLastname(String lastname, Pageable pageable);
Slice<User> findByLastname(String lastname, Pageable pageable);
List<User> findByLastname(String lastname, Sort sort);
List<User> findByLastname(String lastname, Pageable pageable);
```


##### 限制查询结果
案例：在查询方法上加限制查询结果的关键字 First 和 top。
```
User findFirstByOrderByLastnameAsc();
User findTopByOrderByAgeDesc();
Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);
Slice<User> findTop3ByLastname(String lastname, Pageable pageable);
List<User> findFirst10ByLastname(String lastname, Sort sort);
List<User> findTop10ByLastname(String lastname, Pageable pageable);
```
查询方法的结果可以通过关键字来限制 first 或 top，其可以被可互换使用，可选的数值可以追加到顶部/第一个以指定要返回的最大结果的大小。如果数字被省略，则假设结果大小为 1，限制表达式也支持 Distinct 关键字。此外，对于将结果集限制为一个实例的查询，支持将结果包装到一个实例中 Optional。如果将分页或切片应用于限制查询分页（以及可用页数的计算），则在限制结果中应用。


#### 查询结果的不同形式（List/Stream/Page/Future）
Page 和 List 在上面的案例中都有涉及下面将介绍的几种特殊的方式。

##### 流式查询结果
可以通过使用 Java 8 Stream 作为返回类型来逐步处理查询方法的结果，而不是简单地将查询结果包装在 Stream 数据存储中，特定的方法用于执行流。

示例：使用 Java 8 流式传输查询的结果 Stream。
```
@Query("select u from User u")
Stream<User> findAllByCustomQueryAndStream();
Stream<User> findAllByFirstnameNotNull();
@Query("select u from User u")
Stream<User> streamAllPaged(Pageable pageable);
```
注意：流的关闭问题，try cache 是一种用关闭方法。
```
Stream<User> stream;
try {
   stream = repository.findAllByCustomQueryAndStream()
   stream.forEach(…);
} catch (Exception e) {
   e.printStackTrace();
} finally {
   if (stream!=null){
      stream.close();
   }
}
```

##### 异步查询结果
可以使用 Spring 的异步方法执行功能异步执行存储库查询，这意味着方法将在调用时立即返回，并且实际的查询执行将发生在已提交给 Spring TaskExecutor 的任务中，比较适合定时任务的实际场景。
```
//使用 java.util.concurrent.Future 的返回类型。
@Async
Future<User> findByFirstname(String firstname);(1)
//使用 java.util.concurrent.CompletableFuture 作为返回类型。
@Async
CompletableFuture<User> findOneByFirstname(String firstname); (2)
//使用 org.springframework.util.concurrent.ListenableFuture 作为返回类型。
@Async
ListenableFuture<User> findOneByLastname(String lastname);(3)
```


### Projections 对查询结果的扩展
Spring JPA 对 Projections 的扩展的支持，个人觉得这是个非常好的东西，从字面意思上理解就是映射，指的是和 DB 的查询结果的字段映射关系。一般情况下，我们是返回的字段和 DB 的查询结果的字段是一一对应的，但有的时候，需要返回一些指定的字段，不需要全部返回，或者返回一些复合型的字段，还得自己写逻辑。Spring Data 正是考虑到了这一点，允许对专用返回类型进行建模，以便更有选择地将部分视图对象。
假设 Person 是一个正常的实体，和数据表 Person 一一对应，我们正常的写法如下：
```
@Entity
class Person {
   @Id
   UUID id;
   String firstname, lastname;
   Address address;
   @Entity
   static class Address {
      String zipCode, city, street;
   }
}
interface PersonRepository extends Repository<Person, UUID> {
   Collection<Person> findByLastname(String lastname);
}
```
（1）但是我们想仅仅返回其中的 name 相关的字段，应该怎么做呢？如果基于 projections 的思路，其实是比较容易的。只需要声明一个接口，包含我们要返回的属性的方法即可。如下：
```
interface NamesOnly {
  String getFirstname();
  String getLastname();
}
```
Repository 里面的写法如下，直接用这个对象接收结果即可，如下：
```
interface PersonRepository extends Repository<Person, UUID> {
  Collection<NamesOnly> findByLastname(String lastname);
}
```
Ctroller 里面直接调用这个对象可以看看结果。原理是，底层会有动态代理机制为这个接口生产一个实现实体类，在运行时。
（2）查询关联的子对象，一样的道理，如下：
```
interface PersonSummary {
  String getFirstname();
  String getLastname();
  AddressSummary getAddress();
  interface AddressSummary {
    String getCity();
  }
}
interface PersonRepository extends Repository<Person, UUID> {
  Collection<PersonSummary> findByLastname(String lastname);
}
```

（3）@Value 和 SPEL 也支持：
```
interface NamesOnly {
  @Value("#{target.firstname + ' ' + target.lastname}")
  String getFullName();
  …
}
```
PersonRepository 里面保持不变，这样会返回一个 firstname 和 lastname 相加的只有 fullName 的结果集合。
（6）这时候有人会在想，只能用 interface 吗？dto 支持吗？也是可以的，也可以定义自己的 Dto 实体类，需要哪些字段我们直接在 Dto 类当中暴漏出来 get/set 属性即可，如下：
```
class NamesOnlyDto {
  private final String firstname, lastname;
//注意构造方法
  NamesOnlyDto(String firstname, String lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
  }
  String getFirstname() {
    return this.firstname;
  }
  String getLastname() {
    return this.lastname;
  }
}
```

（7）支持动态 Projections，想通过泛化，根据不同的业务情况，返回不通的字段集合。

PersonRepository做一定的变化，如下：
interface PersonRepository extends Repository<Person, UUID> {
  Collection<T> findByLastname(String lastname, Class<T> type);
}
我们的调用方，就可以通过 class 类型动态指定返回不同字段的结果集合了，如下：
```
void someMethod(PersonRepository people) {
//我想包含全字段，就直接用原始entity（Person.class）接收即可
  Collection<Person> aggregates = people.findByLastname("Matthews", Person.class);
//如果我想仅仅返回名称，我只需要指定Dto即可。
  Collection<NamesOnlyDto> aggregates = people.findByLastname("Matthews", NamesOnlyDto.class);
}
```
Projections 的应用场景还是挺多的，望大家好好体会，这样可以实现更优雅的代码，去实现不同的场景。**不必要用数组，冗余的对象去接收查询结果。**













































