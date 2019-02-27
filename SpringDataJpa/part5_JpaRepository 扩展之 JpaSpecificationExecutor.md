## JpaRepository 扩展之 JpaSpecificationExecutor
### JpaSpecificationExecutor 介绍
通过前面的几节内容介绍，我们其实大部分的简单场景可以满足，但是一个 ORM 的解决方案绝不能拘泥与简单方案，而 JpaSpecificationExecutor 是 JPA 2.0 提供的 Criteria API 的使用封装，可以用于动态生成 Query 来满足我们业务中的各种复杂场景。Spring Data JPA 为我们提供了 JpaSpecificationExecutor 接口，只要简单实现 toPredicate 方法就可以实现复杂的查询。

### JpaSpecificationExecutor 使用方法
#### JpaSpecificationExecutor 源码和 API

我们也可以通过 idea 工具详细看其用法和实现类，JpaSpecificationExecutor 是 Repository 要继承的接口，而 SimpleJpaRepository 是其默认实现。而通过源码来看其提供的 API 比较简单、明了，有如下几个方法：
```
public interface JpaSpecificationExecutor<T> {
   //根据 Specification 条件查询单个对象，注意的是，如果条件能查出来多个会报错
   T findOne(@Nullable Specification<T> spec);
   //根据 Specification 条件查询 List 结果
   List<T> findAll(@Nullable Specification<T> spec);
   //根据 Specification 条件，分页查询
   Page<T> findAll(@Nullable Specification<T> spec, Pageable pageable);
   //根据 Specification 条件，带排序的查询结果
   List<T> findAll(@Nullable Specification<T> spec, Sort sort);
   //根据 Specification 条件，查询数量
   long count(@Nullable Specification<T> spec);
}
```

而如果查看 Specifications 源码的话就会发现，其已经将来要被删除了，已经不推荐使用了，而另外两个都是局部私有的，所以真正关注的就是 Specification 接口中如下一个接口方法：
```
public interface Specification<T> {
   Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);
}
```

从这里可以看出，每个调用的地方都需要，创建 Specification 的实现类，而 JpaSpecificationExecutor 是针对 Criteria API 进行了 predicate 标准封装，帮我们封装了通过 EntityManager 的查询和使用细节，使操作 Criteria 更加便利了一些。所以我们要掌握一下 Predicate、Root、CriteriaQuery、CriteriaBuilder 是什么？

### Criteria 的概念简单介绍

**（1）Root root ** 

代表了可以查询和操作的实体对象的根，如果将实体对象比喻成表名，那 **root 里面就是这张表里面的字段**，这不过是 JPQL 的实体字段而已。通过里面的 Path get(String attributeName)，来获得我们想操作的字段。

** （2）CriteriaQuery query ** 

代表一个 specific 的顶层查询对象，它包含着查询的各个部分，比如 select 、from、where、group by、order by 等。CriteriaQuery 对象只对实体类型或嵌入式类型的 Criteria 查询起作用，简单理解，它提供了查询 ROOT 的方法。常用的方法有：
```
CriteriaQuery<T> where(Predicate... restrictions);
CriteriaQuery<T> select(Selection<? extends T> selection);
CriteriaQuery<T> having(Predicate... restrictions);
```

**（3）CriteriaBuilder cb **

用来构建 CritiaQuery 的构建器对象，其实就相当于条件或者是条件组合，并以 Predicate 的形式返回。下面是构建简单的 Predicate 示例：
```
Predicate p1=cb.like(root.get(“name”).as(String.class), “%”+uqm.getName()+“%”);
Predicate p2=cb.equal(root.get("uuid").as(Integer.class), uqm.getUuid());
Predicate p3=cb.gt(root.get("age").as(Integer.class), uqm.getAge());
```

构建组合的 Predicate 示例：
```
Predicate p = cb.and(p3,cb.or(p1,p2));
```

**（4）实际经验**

到此我们发现其实 JpaSpecificationExecutor 帮我提供了一个高级的入口和结构，通过这个入口，可以使用底层 JPA 的 Criteria 所有方法，其实就可以满足了所有业务场景。但在实际工作中，需要注意的是，如果一旦我们写的实现逻辑太复杂，第二个人看不懂时，那一定是有问题的，我要寻找更简单的、更易懂的、更优雅的方式。比如：

- 分页和排序我们就没有比较自己再去实现一遍逻辑，直接用其开放的 Pageable 和 Sort 即可。
- 当我们过多的使用 group 或者 having、sum、count 等内置的 SQL 函数的时候，我们想想就是通过 Specification 实现了逻辑，这种效率真的高吗？是不是数据在其他算好更好？
- 当我们过多的操作 left join 和 inner Join 的链表查询的时候，我们想想，是不是通过数据库的视图（view）更优雅一点？


### JpaSpecificationExecutor 使用案例
新建两个实体:
```
@Entity(name = "UserInfoEntity")
@Table(name = "user_info", schema = "test")
public class UserInfoEntity  implements Serializable {
   @Id
   @Column(name = "id", nullable = false)
   private Integer id;
   @Column(name = "first_name", nullable = true, length = 100)
   private String firstName;
   @Column(name = "last_name", nullable = true, length = 100)
   private String lastName;
   @Column(name = "telephone", nullable = true, length = 100)
   private String telephone;
   @Column(name = "create_time", nullable = true)
   private Date createTime;
   @Column(name = "version", nullable = true)
   private String version;
   @OneToOne(optional = false,fetch = FetchType.EAGER)
   @JoinColumn(referencedColumnName = "id",name = "address_id",nullable = false)
   @Fetch(FetchMode.JOIN)
   private UserReceivingAddressEntity addressEntity;
......
}
```

```
@Entity
@Table(name = "user_receiving_address", schema = "test")
public class UserReceivingAddressEntity  implements Serializable {
   @Id
   @Column(name = "id", nullable = false)
   private Integer id;
   @Column(name = "user_id", nullable = false)
   private Integer userId;
   @Column(name = "address_city", nullable = true, length = 500)
   private String addressCity;
......
}
```

UserRepository 需要继承 JpaSpecificationExecutor
```
public interface UserRepository extends JpaSpecificationExecutor<UserInfoEntity> {
}
```

**调用者 UserInfoManager 的写法**
- 我们演示一下直接用 lambda 使用 Root 和 CriteriaBuilder 做一个简单的不同条件的查询和链表查询。
 <font color=red >可以仔细体会下面这个案例，实际工作中应该大部分都是这种写法，就算扩展也是百变不离其中。</font>
```
@Component
public class UserInfoManager {
   @Autowired
   private UserRepository userRepository;
   
   public Page<UserInfoEntity> findByCondition(UserInfoRequest userParam,Pageable pageable){
      return userRepository.findAll((root, query, cb) -> {
         List<Predicate> predicates = new ArrayList<Predicate>();
         if (StringUtils.isNoneBlank(userParam.getFirstName())){
            //liked的查询条件
            predicates.add(cb.like(root.get("firstName"),"%"+userParam.getFirstName()+"%"));
         }
         if (StringUtils.isNoneBlank(userParam.getTelephone())){
            //equal查询条件
            predicates.add(cb.equal(root.get("telephone"),userParam.getTelephone()));
         }
         if (StringUtils.isNoneBlank(userParam.getVersion())){
            //greaterThan大于等于查询条件
            predicates.add(cb.greaterThan(root.get("version"),userParam.getVersion()));
         }
         if (userParam.getBeginCreateTime()!=null&&userParam.getEndCreateTime()!=null){
            //根据时间区间去查询   predicates.add(cb.between(root.get("createTime"),userParam.getBeginCreateTime(),userParam.getEndCreateTime()));
         }
         if (StringUtils.isNotBlank(userParam.getAddressCity())) {
            //联表查询，利用root的join方法，根据关联关系表里面的字段进行查询。
            predicates.add(cb.equal(root.join("addressEntityList").get("addressCity"), userParam.getAddressCity()));
         }
         return query.where(predicates.toArray(new Predicate[predicates.size()])).getRestriction();
      }, pageable);
   }
}
//可以仔细体会上面这个案例，实际工作中应该大部分都是这种写法，就算扩展也是百变不离其中。
```

- 我们再来看一个不常见的复杂查询的写法，来展示一下 CriteriaQuery 的用法（作者已经强烈不推荐了哦，和上面比起来太不优雅了）。
```
public List<MessageRequest> findByConditions(String name, Integer price, Integer stock) {  
        messageRequestRepository.findAll((Specification<MessageRequest>) (itemRoot, query, criteriaBuilder) -> {
            //这里用 List 存放多种查询条件，实现动态查询
            List<Predicate> predicatesList = new ArrayList<>();
            //name 模糊查询，like 语句
            if (name != null) {
                predicatesList.add(
                    criteriaBuilder.and(
                        criteriaBuilder.like(
                            itemRoot.get("name"), "%" + name + "%")));
            }
            // itemPrice 小于等于 <= 语句
            if (price != null) {
                predicatesList.add(
                    criteriaBuilder.and(
                        criteriaBuilder.le(
                            itemRoot.get("price"), price)));
            }
            //itemStock 大于等于 >= 语句
            if (stock != null) {
                predicatesList.add(
                    criteriaBuilder.and(
                        criteriaBuilder.ge(
                            itemRoot.get("stock"), stock)));
            }
            //where() 拼接查询条件
            query.where(predicatesList.toArray(new Predicate[predicatesList.size()]));
            //返回通过 CriteriaQuery 拼装的 Predicate
            return query.getRestriction();
        });
    }
```

- 而没有 Spring Data JPA 封装之前，如果想获得此三个对象 Root root, CriteriaQuery query, CriteriaBuilder criteriaBuilder，老式 Hibernate 的写法如下（PS：强烈不推荐哦，虽然现在也支持，只是让大家知道了解一下。）：
```
@Autowired //导入entityManager
 private EntityManager entityManager;
//创建CriteriaBuilder安全查询工厂，CriteriaBuilder是一个工厂对象,安全查询的开始.用于构建JPA安全查询.
CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
//创建CriteriaQuery安全查询主语句
//CriteriaQuery对象必须在实体类型或嵌入式类型上的Criteria 查询上起作用。
CriteriaQuery<Item> query = criteriaBuilder.createQuery(Item.class);
//Root 定义查询的From子句中能出现的类型
Root<Item> itemRoot = query.from(Item.class);
```

- 我们再来看一个利用 CriteriaQuery 例子，其实大家可以扩展一下思路，就是 **Hibernate 那套在这里面都支持，不过作者还是建议代码越简单越好**。
```
List<UserSpuFavoriteEntity> result = userSpuFavoriteDao.findAll((Root<UserSpuFavoriteEntity> root, CriteriaQuery<?> query, CriteriaBuilder cb)->{
	query.where(cb.and(cb.equal(root.get("userName"), userName),cb.isFalse(root.get("isDelete"))));
	query.orderBy(cb.desc(root.get("updateTime")));
	return query.getRestriction();
});
```

### Specification 工作中的一些扩展
我们在实际工作中会发现，如果上面的逻辑，简单重复写总感觉是不是可以抽出一些公用方法呢，此时引入一种工厂模式，帮我们做一些事情，可以让代码更加优雅。基于 JpaSpecificationExecutor 的思路，我们创建一个 SpecificationFactory.Java 内容如下：

```
public final class SpecificationFactory {
   /**
    * 模糊查询，匹配对应字段
    */
   public static Specification containsLike(String attribute, String value) {
      return (root, query, cb)-> cb.like(root.get(attribute), "%" + value + "%");
   }
   /**
    * 某字段的值等于 value 的查询条件
    */
   public static Specification equal(String attribute, Object value) {
      return (root, query, cb) -> cb.equal(root.get(attribute),value);
   }
   /**
    * 获取对应属性的值所在区间
    */
   public static Specification isBetween(String attribute, int min, int max) {
      return (root, query, cb) -> cb.between(root.get(attribute), min, max);
   }
   public static Specification isBetween(String attribute, double min, double max) {
      return (root, query, cb) -> cb.between(root.get(attribute), min, max);
   }
   public static Specification isBetween(String attribute, Date min, Date max) {
      return (root, query, cb) -> cb.between(root.get(attribute), min, max);
   }
   /**
    * 通过属性名和集合实现 in 查询
    */
   public static Specification in(String attribute, Collection c) {
      return (root, query, cb) ->root.get(attribute).in(c);
   }
   /**
    * 通过属性名构建大于等于 Value 的查询条件
    */
   public static Specification greaterThan(String attribute, BigDecimal value) {
      return (root, query, cb) ->cb.greaterThan(root.get(attribute),value);
   }
   public static Specification greaterThan(String attribute, Long value) {
      return (root, query, cb) ->cb.greaterThan(root.get(attribute),value);
   }
......
}
```

**调用实例1：**
```
userRepository.findAll(
      SpecificationFactory.containsLike("firstName", userParam.getLastName()),
      pageable);
```
是不是发现代码一下子少了很多？

配合 Specifications 使用，调用实例2：
```
userRepository.findAll(Specifications.where(
      SpecificationFactory.containsLike("firstName", userParam.getLastName()))
            .and(SpecificationFactory.greaterThan("version",userParam.getVersion())),
      pageable);
```
和我们前面举的例子比起来是不是代码更加优雅、可读性更加强了？


### JpaSpecificationExecutor 实现原理
我们还是先通过开发工具，把关键的类添加到Diagram上面进行分析，如图：

![在这里插入图片描述](http://images.gitbook.cn/b22ffc50-397b-11e8-9f33-a7686c32808c)

**与 EntityManager 的关系图**

![在这里插入图片描述](http://images.gitbook.cn/f0cc4580-3986-11e8-9b9a-53206b7f130d)






























