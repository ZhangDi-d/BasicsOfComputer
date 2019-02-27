## 注解式查询方法
声明式的查询方法，即注解的查询用法大全，这种也是平时工作中最常见的用法.
![在这里插入图片描述](http://images.gitbook.cn/b7bad320-2dcd-11e8-929f-9f2bcc393f42)
### @Query 详解
#### 语法及其源码
```
public @interface Query {
   /**
    * 指定JPQL的查询语句。（nativeQuery=true的时候，是原生的Sql语句）
    */
   String value() default "";
   /**
    * 指定count的JPQL语句，如果不指定将根据query自动生成。
    * （如果当nativeQuery=true的时候，指的是原生的Sql语句）
    */
   String countQuery() default "";
   /**
    * 根据哪个字段来count，一般默认即可。
    */
   String countProjection() default "";
   /**
    * 默认是false，表示value里面是不是原生的sql语句
    */
   boolean nativeQuery() default false;
   /**
    * 可以指定一个query的名字，必须唯一的。
    * 如果不指定，默认的生成规则是：
    * {$domainClass}.${queryMethodName}
    */
   String name() default "";
   /*
    * 可以指定一个count的query的名字，必须唯一的。
    * 如果不指定，默认的生成规则是：
    * {$domainClass}.${queryMethodName}.count
    */
   String countName() default "";
}
```

#### @Query 用法
使用命名查询为实体声明查询是一种有效的方法，对于少量查询很有效。一般只需要关心 @Query 里面的 value 和 nativeQuery 的值。使用声明式 JPQL 查询有个好处，就是启动的时候就知道你的语法正确不正确。

>注意：好的架构师写代码时报错的顺序是编译<启动<运行时，即越早发现错误越好。默认 value 里面是 JPQL 语法，既对象查询和 SQL、HQL 比较类似。

案例 4.1：声明一个注解在 Repository 的查询方法上。
```
public interface UserRepository extends JpaRepository<User, Long>{
  @Query("select u from User u where u.emailAddress = ?1")
  User findByEmailAddress(String emailAddress);
}
```

案例 4.2：Like 查询，注意 **firstname 不会自动加上 % 关键字**。
```
public interface UserRepository extends JpaRepository<User, Long> {
  @Query("select u from User u where u.firstname like %?1")
  List<User> findByFirstnameEndsWith(String firstname);
}
```

案例 4.3：直接用原始 SQL。
```
public interface UserRepository extends JpaRepository<User, Long> {
  @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
  User findByEmailAddress(String emailAddress);
}
```

>注意：nativeQuery 不支持直接 Sort 的参数查询。
案例4.4：nativeQuery 的排序错误的写法，下面这个是启动不起来的。
```
public interface UserRepository extends JpaRepository<User, Long> {
@Query(value = "select * from user_info where first_name=?1",nativeQuery = true)
List<UserInfoEntity> findByFirstName(String firstName,Sort sort);
}
```

案例4.5：nativeQuery 排序的写法。
```
@Query(value = "select * from user_info where first_name=?1 order by ?2",nativeQuery = true)
List<UserInfoEntity> findByFirstName(String firstName,String sort);
//调用的地方写法last_name是数据里面的字段名，不是对象的字段名
repository.findByFirstName("jackzhang","last_name");
```



#### @Query 的排序
@Query 的 JPQL 情况下，想实现排序，方法上面直接用 PageRequest 或者直接用 Sort 参数都可以做到。

在排序实例中实际使用的属性需要与实体模型里面的字段相匹配，这意味着它们需要解析为查询中使用的属性或别名。这是一个state_field_path_expression JPQL定义，并且 Sort 的对象支持一些特定的函数。

案例 4.6：Sort and JpaSort 的使用。
```
public interface UserRepository extends JpaRepository<User, Long> {
  @Query("select u from User u where u.lastname like ?1%")
  List<User> findByAndSort(String lastname, Sort sort);
  @Query("select u.id, LENGTH(u.firstname) as fn_len from User u where u.lastname like ?1%")
  List<Object[]> findByAsArrayAndSort(String lastname, Sort sort);
}
//调用方的写法，如下：
repo.findByAndSort("lannister", new Sort("firstname"));               
repo.findByAndSort("stark", new Sort("LENGTH(firstname)"));          
repo.findByAndSort("targaryen", JpaSort.unsafe("LENGTH(firstname)"));
repo.findByAsArrayAndSort("bolton", new Sort("fn_len"));   
```

#### @Query 的分页
案例 4.7：直接用 Page 对象接受接口，参数直接用 Pageable 的实现类即可。
```
public interface UserRepository extends JpaRepository<User, Long> {
  @Query(value = "select u from User u where u.lastname = ?1")
  Page<User> findByLastname(String lastname, Pageable pageable);
}
//调用者的写法
repository.findByFirstName("jackzhang",new PageRequest(1,10));
```

案例 4.8：对原生 SQL 的分页支持，案例如下，但是支持的不是特别友好，以 MySQL 为例。
```
 public interface UserRepository extends JpaRepository<UserInfoEntity, Integer>, JpaSpecificationExecutor<UserInfoEntity> {
   @Query(value = "select * from user_info where first_name=?1 /* #pageable# */",
         countQuery = "select count(*) from user_info where first_name=?1",
         nativeQuery = true)
   Page<UserInfoEntity> findByFirstName(String firstName, Pageable pageable);
}
//调用者的写法
return userRepository.findByFirstName("jackzhang",new PageRequest(1,10, Sort.Direction.DESC,"last_name"));
//打印出来的sql
select  *   from  user_info  where  first_name=? /* #pageable# */  order by  last_name desc limit ?, ?
```

注意：

这个注释 /* #pageable# */ 必须有；
估计有可能随着版本的变化这个会做优化。
另外一种实现方法就是自己写两个查询方法，自己手动分页。


### @Param 用法
默认情况下，参数是通过顺序绑定在查询语句上的，这使得查询方法对参数位置的重构容易出错。为了解决这个问题，可以使用 @ Param 注解指定方法参数的具体名称，通过绑定的参数名字做查询条件，这样不需要关心参数的顺序，推荐这种做法，比较利于代码重构。

案例 4.9：根据参数进行查询。

public interface UserRepository extends JpaRepository<User, Long> {
  @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
  User findByLastnameOrFirstname(@Param("lastname") String lastname,
                                 @Param("firstname") String firstname);
}

案例 4.10：根据参数进行查询，top 10 前面说的 query method 关键字照样有用，如下：
```
public interface UserRepository extends JpaRepository<User, Long> {
  @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
  User findTop10ByLastnameOrFirstname(@Param("lastname") String lastname,
                                 @Param("firstname") String firstname);
}
```
提醒：大家通过 @Query 定义自己的查询方法时，建议也用 Spring Data JPA 的 name query 的命名方法，这样下来风格就比较统一了。


### Spel 表达式的支持
在 Spring Data JPA 1.4 以后，支持在 @Query 中使用 SpEL 表达式（简介）来接收变量。

|变量名|  使用方式| 描述 |
|--|--|--|
|entityName  | select x from #{#entityName} x | 根据指定的 Repository 自动插入相关的 entityName|

>有两种方式能被解析出来：
如果定了 @Entity 注解，直接用其属性名。
如果没定义，直接用实体的类的名称。

在以下的例子中，我们在查询语句中插入表达式：
```
@Entity("User")
public class User {
   @Id
   @GeneratedValue
   Long id;
   String lastname;
}

//Repository写法
public interface UserRepository extends JpaRepository<User, Long> {
   @Query("select u from #{#entityName} u where u.lastname = ?1")
   List<User> findByLastname(String lastname);
}
```


### @Modifying 修改查询
学习思路一样，我们先看源码：
```
public @interface Modifying {
//如果配置了一级缓存，这个时候用clearAutomatically=true,就会刷新hibernate的一级缓存了， 不然你在同一接口中，更新一个对象，接着查询这个对象，那么你查出来的这个对象还是之前的没有更新之前的状态。这个比较适合老Hibernate的开发的程序员。
    boolean clearAutomatically() default false;
}
```

可以通过在 @Modifying 注解实现只需要参数绑定的 update 查询的执行，我们来看个例子根据 lastName 更新 firstname 并且返回更新条数如下：
```
@Modifying
@Query("update User u set u.firstname = ?1 where u.lastname = ?2")
int setFixedFirstnameFor(String firstname, String lastname);
```
简单的针对某些特定属性的更新，也可以直接用基类里面提供的通用 save 来做更新（即继承 CrudRepository 接口）。

**还有第三种方法就是自定义 Repository 使用 EntityManager 来进行更新操作。**
对删除操作的支持如下：
```
interface UserRepository extends Repository<User, Long> {
  void deleteByRoleId(long roleId);
  @Modifying
  @Query("delete from User u where user.role.id = ?1")
  void deleteInBulkByRoleId(long roleId);
}
```
所以现在我们一共有四种方式来做更新操作：

1.通过方法表达式（method name query），上篇内容已介绍；
2.还有一种就是 @Modifying 注解；
3.@Query 注解也可以做到；
4.继承 CrudRepository 接口。


### @QueryHints
有很多数据库支持 Hint Query 的语法，不过这种查询支持比较老旧，感觉应该会慢慢被淘汰，工作中很少见有人使用。但 Spring Data JPA 还是做了很好的支持，它只支持一些固定的 HintValue 值，用来优化 Query 的作用。有两个注解需要了解 @QueryHints、value 等于多个 @QueryHint。

### @Procedure 储存过程的查询方法
>我们通过 @Procedure 来介绍一下，JPA 对储存过程的支持。

1）@Procedure 源码如下：
```
public @interface Procedure {
   // 数据库里面储存过程的名称
   String value() default "";
   // 数据库里面储存过程的名称
   String procedureName() default "";
   //在EntityManager中的名字，NamedStoredProcedureQuery使用
   String name() default "";
   //输出参数的名字
   String outputParameterName() default "";
}
```

（2）首先创建一个储存过程名字叫 plus1inout 有两个参数、两个结果。
```
CREATE PROCEDURE plus1inout(IN arg int, OUT res int)
BEGIN
 SELECT (arg+10) into res;
END
```

（3）我们可以使用 @NamedStoredProcedureQueries 注释来调用存储过程，这个必须定义在一个实体上面。
```
@Entity
@NamedStoredProcedureQuery(name = "User.plus1", procedureName = "plus1inout", parameters = {
@StoredProcedureParameter(mode = ParameterMode.IN, name = "arg", type = Integer.class),
@StoredProcedureParameter(mode = ParameterMode.OUT, name = "res", type = Integer.class) })
public class User {
   //这个是一个Procedure实体类，可以通过NamedStoredProcedureQueries在这个类里面定义多个储存过程的查询。
}
```
关键要点：
- 存储过程使用了注释 @NamedStoredProcedureQuery，并绑定到一个 JPA 表；
- procedureName 是存储过程的名字；
- name 是 JPA 中的存储过程的名字；
- 使用注释 @StoredProcedureParameter 来定义存储过程使用的 IN/OUT 参数。
（4）直接通过自定义过的 Repository 完成储存过程的调用。
```
public interface MyUserRepository extends CrudRepository<User, Long> {
@Procedure("plus1inout")//通过储存过程的名字
Integer explicitlyNamedPlus1inout(Integer arg);
@Procedure(procedureName = "plus1inout")//通过储存过程的名字
Integer plus1inout(Integer arg);
@Procedure(name = "User.plus1IO")//自定义的储存过程的名字
Integer entityAnnotatedCustomNamedProcedurePlus1IO(@Param("arg") Integer arg);
}
```

关键要点：

- @Procedure 的 procedureName 参数必须匹配 @NamedStoredProcedureQuery 的 procedureName。
- @Procedure 的 name 参数必须匹配 @NamedStoredProcedureQuery 的 name。
- @Param 必须匹配 @StoredProcedureParameter 注释的 name 参数。
- 返回类型必须匹配：in_only_test存储过程返回是 void，in_and_out_test存储过程必须返回 String。


### @NamedQueries 预定义查询
#### 这种是预定义查询的一种形式

（1）在 @Entity 下增加 @NamedQuery 定义。
```
public @interface NamedQuery {
   //query的名称，规则：实体.方法名；
   String name();
   //具体的JPQL查询语法
   String query();
}
```
>需要注意，这里的 Query 里面的值也是 JPQL，查询参数也要和实体进行对应起来。因为实际场景 中这种破坏 Entity 的侵入式**很不美感，也不方便**，所以这种方式容易遗忘，工作中也很少推荐的一种方式。

（2）与之相对应的还有 @NamedNativeQuery。用法一样，唯一不一样的是，Query 里面放置的是原生 SQL 语句，而非实体的字段名字。

##### 用法举例
（1）实体里面的写法。
```
@Entity
@NamedQuery(name="Customer.findByFirstName",query = "select c from Customer c where c.firstName = ?1")
public class Customer {
   @Id
   @GeneratedValue(strategy = GenerationType.AUTO)
   private Long id;
   private String firstName;
   private String lastName;
......
}
```

（2）CustomerRepository 里面的写法。
```
Customer findByFirstName(String bauer);
```

（3）调用者的写法。
```
Customer customer = repository.findByFirstName("Bauer");
@NamedQuery 和 @Query 方法定义查询三者对比：
```
Spring JPA 里面的有先级，咱们前面章节有讲到过：@Query > @NameQuery > 方法定义查询。
推荐使用的有优先级：@Query > 方法定义查询 > @NameQuery。
相同点，都不支持动态条件查询。
































