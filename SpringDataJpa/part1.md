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





