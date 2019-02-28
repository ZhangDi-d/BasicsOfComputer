## JpaRepository 扩展之自定义 Repository
### 自定义 JpaRepository 简介
由于业务场景的千差万别，有可能需要定义自己的 Repository 类，其实通过上面的章节，我们也大概能想到 Spring Data JPA 可以**轻松地允许你提供自定义 Repository**，并且还很容易与现有的抽象和查询方法集成。

> 本章节内容将详细介绍一下如何自定义以及在实际工作中作者做了哪些自定义方法。

### EntityManager 介绍
我们前面已经无数次提到了，JPA 的默认 Repository 的实现类是 SimpleJpaRepository，而里面的具体实现就是调用的 EntityManager。对于 javax.persistence.EntityManager 通过源码，先来看下它主要给我们提供了哪几个方法： 
```
public interface EntityManager {
  /**
    *根据主键查询实体对象
    */
  public <T> T find(Class<T> entityClass, Object primaryKey);
    /**
     *  支持JQPL的语法
     * @param qlString a Java Persistence query string
     */
    public Query createQuery(String qlString);
    /**
     * 利用CriteriaQuery来创建查询
     * @param criteriaQuery  a criteria query object
     */
    public <T> TypedQuery<T> createQuery(CriteriaQuery<T> criteriaQuery);
    /**
     * 利用CriteriaUpdate创建更新查询
     * @param updateQuery a criteria update query object
     */
    public Query createQuery(CriteriaUpdate updateQuery);
    /**
     * 利用CriteriaDelete创建删除查询
     * @param deleteQuery a criteria delete query object
     */
    public Query createQuery(CriteriaDelete deleteQuery);
    /**
     * 利用原生的sql语句创建查询，可以是查询、更新、删除等sql
     * @param sqlString a native SQL query string
     */
    public Query createNativeQuery(String sqlString);
    /**
     * 利用原生SQL查询，指定返回结果类型
     * @param sqlString a native SQL query string
     * @param resultClass the class of the resulting instance(s)
     */
    public Query createNativeQuery(String sqlString, Class resultClass);
......
}
```

而 javax.persistence.EntityManager 就是 Java Persitence API 的核心操作方法了，我们可以在 SimpleJpaRepository 的构造方法上面设置一断点，如下位置，就可以发现 entityManager 是由 [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@2dac78aa] 动态代理管理，而动态代理的实现类就是 org.hibernate.internal.SessionImpl，这就是 Spring Data JPA 的核心封装和实现了（如果是从 Hibernate 切换过来的，这部分内容就不陌生了）。
```
/**
 * Creates a new {@link SimpleJpaRepository} to manage objects of the given {@link JpaEntityInformation}.
 *
 * @param entityInformation must not be {@literal null}.
 * @param entityManager must not be {@literal null}.
 */
public SimpleJpaRepository(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
	Assert.notNull(entityInformation, "JpaEntityInformation must not be null!");
	Assert.notNull(entityManager, "EntityManager must not be null!");
	this.entityInformation = entityInformation;
	this.em = entityManager;
	this.provider = PersistenceProvider.fromEntityManager(entityManager);
}
```

### EntityManager 的简单使用案例
案例1：针对复杂的原生 SQL 的查询
```
/创建sql语句
StringBuilder querySQL = new StringBuilder("SELECT spu_id AS spuId ,spu_name AS spuName,")
        .append("SUM(system_price_count) AS systemPriceCount,")
        .append("SUM(wechat_applet_view_count) AS wechatAppletViewCount")
        .append(" FROM report_spu_summary ");
//利用entityManager实现查询
Query query = entityManager.createNativeQuery(querySQL.toString() + whereSQL.toString() + groupBy + orderBy.toString());
//分页
query.setFirstResult(custom.offset()).setMaxResults(custom.getPageSize());
//结果转换
query.unwrap(SQLQuery.class).setResultTransformer(Transformers.aliasToBean(ReportSpuSummarySumBo.class));
//得到最终的返回结果
List<ReportSpuSummarySumBo> results = query.getResultList();
```
此案例仅仅为了说明 entityManager.createNativeQuery 的查询方法，但**是不推荐用这种用法**，开发思路可转换一下，做到心中有数即可。

案例2：find 方法
```
entityManager.find(UserInfoEntity.class,1);
```
案例3：JPQL 的用法
```
Query query = entityManager.createQuery("SELECT c FROM Customer c");  
List<Customer> result = query.getResultList();  
```

### 自定义实现 Repository
#### EntityManager 的获取方式
我们既然要自定义，首先讲一下 EntityManager 的两种获取方式。

**1. 通过 @PersistenceContext 注解。**
通过将 @PersistenceContext 注解标注在 EntityManager 类型的字段上，这样得到的 EntityManager 就是容器管理的 EntityManager。由于是容器管理的，所以我们不需要也不应该显式关闭注入的 EntityManager 实例。
```
@Repository
@Transactional(readOnly = true)
public class UserRepositoryImpl implements UserRepositoryCustom {
    @PersistenceContext  //获得entityManager的实例
    EntityManager entityManager;
}
```
2. 继承 SimpleJpaRepository 成为子类，实现构造方法即可，这时候我们直接用父类里面的 EntityManager 即可。
public class BaseRepositoryCustom<T, ID> extends SimpleJpaRepository<T, ID> {
    public BaseRepositoryCustom(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
        super(entityInformation, entityManager);
    }
    public BaseRepositoryCustom(Class<T> domainClass, EntityManager em) {
        super(domainClass, em);
    }
}


### 自定义 Repository 的两种场景
我们自定义实现 Repository，主要的应用场景有两种：

- 个别特殊化场景，私有的。
- 公用的通用的场景，替代默认的 SimpleJpaRepository 的场景，架构层面出发。

**1. 自定义个别的特殊场景，私有的 Repository。**
这种方法就是需要自己创建一个接口，和对应的接口实现类，若需要用到特殊化的实现方法的话，***Respository 只需要继承你自定义的接口即可，主要有两种能力：
- 实现自定义接口
- 可以直接覆盖 Spring Data JPA 给我们提供的默认 Respository 的接口里面的方法。

案例1：单个私有的 Repository 接口实现类
（1）创建自定义接口
```
/**
 * @author jack
 */
public interface UserRepositoryCustom {
    /**
     * 自定义一个查询方法，name的like查询，此处仅仅是演示例子，实际中直接用QueryMethod即可
     * @param firstName
     * @return
     */
    List<User> customerMethodNamesLike(String firstName);
}
```

（2）自定义存储库功能的实现
```
/**
 * 用@Repository 将此实现交个Spring bean加载
 * 咱们模仿SimpleJpaRepository 默认将所有方法都开启一个事务
 */
@Repository
@Transactional(readOnly = true)
public class UserRepositoryCustomImpl implements UserRepositoryCustom {
    @PersistenceContext
    EntityManager entityManager;
    /**
     * 自定义一个查询firstname的方法
     * @param firstName
     * @return
     */
    @Override
    public List<User> customerMethodNamesLike(String firstName) {
        Query query = entityManager.createNativeQuery("SELECT u.* FROM user as u " +
                "WHERE u.name LIKE ?", User.class);
        query.setParameter(1, firstName + "%");
        return query.getResultList();
    }
}
```
> 我们这里采用 entityManager，当然了也不排除自己通过**最底层的 JdbcTemplate** 来自己实现逻辑。


（3）由于这个接口是为 User 单独写的，但是同时也可以继承和 @Repository 的任何子类。
```
/**
 * 使用的时候直接继承 UserRepositoryCustom接口即可
 */
public interface UserRepository extends Repository<User, Long>,UserRepositoryCustom {
}
```
Controller 的调用方式如下：
```
/**
 * 调用我们自定义的实现方法
 *
 * @return
 */
@GetMapping(path = "/customer")
@ResponseBody
public Iterable<User> findCustomerMethodNamesLike() {
    return userRepository.customerMethodNamesLike("jack");
}
```

（4）其实通过上述方法我们可以实现多个自定义接口：
```
//如：我们自定义了HumanCustomerRepository, ContactCustomerRepository两个Repository
interface UserRepository extends CrudRepository<User, Long>, HumanCustomerRepository, ContactCustomerRepository {
  // 用的时候只需要继承多个自定义接口即可
}
```

（5）覆盖 JPA 里面的默认实现方法

Spring Data JPA 的底层实现里面，**自定义的 Repositories 的实现类和方法要高于它帮我们提供的 Repositories**，所以当我们有场景需要覆盖默认实现的时候其 demo 如下：
```
//假设我们要覆盖默认的save方法的逻辑
interface CustomizedSave<T> {
  <S extends T> S save(S entity);
}
class CustomizedSaveImpl<T> implements CustomizedSave<T> {
  public <S extends T> S save(S entity) {
    // Your custom implementation
  }
}
//用法保持不变，如下：
interface UserRepository extends CrudRepository<User, Long>, CustomizedSave<User> {
}
//CustomizedSave通过泛化可以被多个Repository使用
interface PersonRepository extends CrudRepository<Person, Long>, CustomizedSave<Person> {
}
```

实际工作中应用于逻辑删除场景：

> 在实际工作的生产环境中，我们可能经常会用到逻辑删除，所以做法是一般自定义覆盖 Data JPA 帮我们提供 remove 方法，然后实现逻辑删除的逻辑即可。


**2：公用的通用的场景，替代默认的 SimpleJpaRepository 的场景，从架构层面出发。**
案例2：定义一个公用的 Repository 接口的实现类。
通过构造方法获得 EntityManager，需要用到 Java 的泛化技术。当你想将一个方法添加到所有的存储库接口时，上述方法是不可行的，要将自定义行为添加到所有存储库，首先添加一个中间接口来声明共享行为。
（1）声明定制共享行为的接口，用 @NoRepositoryBean：
```
//因为要公用，所以必须要通用，不能失去本身的Spring Data JPA给我们提供的默认方法，所有我们继承相关的Repository类
@NoRepositoryBean
public interface MyRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID> {
  void sharedCustomMethod(ID id);
}
```

（2）继承 SimpleJpaRepository 扩展自己的方法实现逻辑：
```
public class MyRepositoryImpl<T, ID extends Serializable>
  extends SimpleJpaRepository<T, ID> implements MyRepository<T, ID> {
  private final EntityManager entityManager;
  public MyRepositoryImpl(JpaEntityInformation entityInformation, EntityManager entityManager) {
    super(entityInformation, entityManager);
    // Keep the EntityManager around to used from the newly introduced methods.
    this.entityManager = entityManager;
  }
  public void sharedCustomMethod(ID id) {
    // 通过entityManager实现自己的额外方法的实现逻辑。这里不多说了
  }
}
```

>注意：该类需要具有专门的存储库工厂实现使用超级类的构造函数，如果存储库基类有多个构造函数，则覆盖一个 EntityInformation 加上特定于存储的基础架构对象（例如，一个 EntityManager 或一个模板类），也可以重写 SimpleJpaRepository 的任何逻辑。如逻辑删除放在这里面实现，就不要所有的 Repository 去关心实现哪个接口了。

（3）使用 **JavaConfig 配置自定义 MyRepositoryImpl 作为其他接口的动态代理的实现基类。**

具有全局的性质，即使没有继承它所有的动态代理类也会变成它。
```
@Configuration
@EnableJpaRepositories(repositoryBaseClass = MyRepositoryImpl.class)
class ApplicationConfiguration { … }
```

（4）使用的时候就可以显示的选择用哪个接口，从而选择性的暴露 SimpleJpaRepository 的实现方法。

现在，各个存储库接口将扩展此中间接口，而不是扩展 Repository 接口以包含声明的功能。接下来，创建扩展了持久性技术特定的存储库基类的中间接口的实现。然后，该类将用作存储库代理的自定义基类。
```
//如果你要使用你自定义的全局MyRepositoryImpl只需要继承接口即可，如下：
interface PersonRepository extends MyRepositoryImpl<Person, Long>{
}
```

### 实际工作的应用场景总结及其案例
在实际工作中，有哪些场景会用到自定义 Repository 呢，这里列出几种实际在工作中的应用案例。

#### 1. 逻辑删除场景 (逻辑删除而非物理删除)
可以用到上面说的两种实现方式，如果有框架级别的全局自定义 Respository 那就在全局实现里面覆盖默认 remove 方法，这样就会统一全部只能使用逻辑删除。但是一般是自定义一个特殊的删除Respository，让大家去根据不同的domain业务逻辑去选择使用此接口即可。

#### 2. 当有业务场景要覆盖 SimpleJpaRepository 默认实现的时候
这种一般是具体情况具体分析的，一般实现特殊化的自定义 Respository 即可。












































































































