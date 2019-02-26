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