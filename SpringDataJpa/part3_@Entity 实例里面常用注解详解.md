## @Entity 实例里面常用注解详解

### javax.persistence 概况介绍
虽然 Spring Data JPA 已经对数据的操作封装的很好了，约定大于配置的思想，帮我们默认了很多东西。JPA（Java 持久性 API）是存储业务实体关联的实体的来源，它显示了如何定义一个面向普通 Java 对象（POJO）作为一个实体，以及如何与管理关系实体，提供了一套标准。因此，javax.persistence 下面的有些注解我们还是必须要去了解的，便于更好的提高工作效率。

![在这里插入图片描述](http://images.gitbook.cn/8b0145f0-2de4-11e8-bc9e-6f65679a3102)

|单元|描述|
|---|---|
|EntityManagerFactory|	这是一个 EntityManager 的工厂类，它创建并管理多个 EntityManager 实例|
|EntityManager|	这是一个接口，它管理的持久化操作的对象，它的工作原理类似工厂的查询实例|
|Entity|	实体是持久性对象，是存储在数据库中的记录|
|EntityTransaction|	它与 EntityManager 是一对一的关系，对于每一个 EntityManager，操作是由 EntityTransaction 类维护|
|Persistence|	这个类包含静态方法来获取 EntityManagerFactory 实例|
|Query|	该接口由每个 JPA 供应商，能够获得符合标准的关系对象|


下面我们主要介绍一下，在 Entity 里面常用的注解有哪些，还有很多没有介绍到的，可以直接到包的源码里面进行查找和分析。

### 基本注解 @Entity、@Table、@Id、@GeneratedValue、@Basic、@Column、@Transient、@Lob、@Temporal
先看一个 Blog 的案例其中实体的配置如下：
```
@Entity	//@Entity 用于定义对象将会成为被 JPA 管理的实体，将字段映射到指定的数据库表中
@Table(name = "user_blog", schema = "test") //@Table 用于指定数据库的表名
public class UserBlogEntity {
   @Id	//@Id 定义属性为数据库的主键，一个实体里面必须有一个，并且必须和 @GeneratedValue 配合使用和成对出现。
   @Column(name = "id", nullable = false)
   @GeneratedValue(strategy = GenerationType.IDENTITY,generator="select seq_user_blog.nextval from dual")  //
   private Integer id;
   @Column(name = "title", nullable = true, length = 200)
   private String title;
   @Basic
   @Column(name = "create_user_id", nullable = true)
   private Integer createUserId;
   @Basic
   @Column(name = "blog_content", nullable = true, length = -1)
   @Lob
   private String blogContent;
   @Basic(fetch = FetchType.LAZY)
   @Column(name = "image", nullable = true)
   @Lob
   private byte[] image;
   @Basic
   @Column(name = "create_time", nullable = true)
   @Temporal(TemporalType.TIMESTAMP)
   private Date createTime;
   @Basic
   @Column(name = "create_date", nullable = true)
   @Temporal(TemporalType.DATE)
   private Date createDate;
   @Transient
   private String transientSimple;
......
}
```

（1）@Entity 用于定义对象将会成为被 JPA 管理的实体，将字段映射到指定的数据库表中，源码如下：
```
public @interface Entity {
   //可选，默认是次实体类的名字，全局唯一。
   String name() default "";
}
```

（2）@Table 用于指定数据库的表名：
```
public @interface Table {
   //表的名字，可选。如果不填写，系统认为好实体的名字一样为表名。
   String name() default "";
   //此表的catalog，可选
   String catalog() default "";
   //此表所在schema，可选
   String schema() default "";
   //唯一性约束，只有创建表的时候有用，默认不需要。
   UniqueConstraint[] uniqueConstraints() default { };
   //索引，只有创建表的时候使用，默认不需要。
   Index[] indexes() default {};
}
```
（3）@Id 定义属性为数据库的主键，一个实体里面必须有一个，并且必须和 @GeneratedValue 配合使用和成对出现。

（4）@IdClass 利用外部类的联合主键，源码：
```
public @interface IdClass {
//联合主键的类
    Class value();
}
```

作为符合主键类，要满足以下几点要求。

- **必须实现 Serializable 接口**。
- **必须有默认的 public 无参数的构造方法**。
- **必须覆盖 equals 和 hashCode 方法**。equals 方法用于判断两个对象是否相同，EntityManger 通过 find 方法来查找 Entity 时，是根据 equals 的返回值来判断的。本例中，只有对象的 name 和 email 值完全相同时或同一个对象时则返回 true，否则返回 false。hashCode 方法返回当前对象的哈希码，生成 hashCode 相同的概率越小越好，算法可以进行优化。

（5）@IdClass 用法

1）假设 UserBlog 的联合主键是 createUserId 和 title，新增一个 UserBlogKey 的类。

UserBlogKey.class
```
import java.io.Serializable;
public class UserBlogKey implements Serializable {
   private String title;
   private Integer createUserId;
   public UserBlogKey() {
   }
   public UserBlogKey(String title, Integer createUserId) {
      this.title = title;
      this.createUserId = createUserId;
   }
.....//get set 方法我们略过
}
```
2）UserBlogEntity.java 稍加改动，实体类上需要加 @IdClass 注解和两个主键上都得加 @Id 注解，如下。
```
@Entity
@Table(name = "user_blog", schema = "test")
@IdClass(value = UserBlogKey.class)
public class UserBlogEntity {
   @Column(name = "id", nullable = false)
   private Integer id;
   @Id //两个主键都需要加上@Id
   @Column(name = "title", nullable = true, length = 200)
   private String title;
   @Id //两个主键都需要加上@Id
   @Column(name = "create_user_id", nullable = true)
   private Integer createUserId;
......//不变的部分我们省略
}
```

3）UserBlogRepository 我们做的改动：
```
public interface UserBlogRepository extends JpaRepository<UserBlogEntity,UserBlogKey>{
}
```

4）使用的时候：
```
@RequestMapping(path = "/blog/{title}/{createUserId}")
@ResponseBody
public Optional<UserBlogEntity> showBlogs(@PathVariable(value = "createUserId") Integer createUserId,@PathVariable("title") String title) {
   return userBlogRepository.findById(new UserBlogKey(title,createUserId));
}
```

（6）@GeneratedValue 主键生成策略：
```
public @interface GeneratedValue {
    //Id的生成策略
    GenerationType strategy() default AUTO;
    //通过Sequences生成Id,常见的是Orcale数据库ID生成规则，这个时候需要配合@SequenceGenerator使用
    String generator() default "";
}
```

GenerationType 一共有以下四个值：
```
public enum GenerationType {
    //通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。
    TABLE,
    //通过序列产生主键，通过 @SequenceGenerator 注解指定序列名， MySql 不支持这种方式；
    SEQUENCE,
    //采用数据库ID自增长， 一般用于mysql数据库
    IDENTITY,
	//JPA 自动选择合适的策略，是默认选项；
    AUTO
}
```

（7）@Basic 表示属性是到数据库表的字段的映射。**如果实体的字段上没有任何注解，默认即为 @Basic。**
```
public @interface Basic {
    //可选，EAGER（默认）：立即加载；LAZY：延迟加载。（LAZY主要应用在大字段上面）
    FetchType fetch() default EAGER;
    //可选。这个字段是否可以为null，默认是true。
    boolean optional() default true;
}
```

（8）@Transient 表示该属性并非一个到数据库表的字段的映射，表示非持久化属性。JPA 映射数据库的时候忽略它，与 @Basic 相反的作用。

（9）@Column 定义该属性对应数据库中的列名。
```
public @interface Column {
    //数据库中的表的列名；可选，如果不填写认为字段名和实体属性名一样。
    String name() default "";
    //是否唯一。默认flase，可选。
    boolean unique() default false;
    //数据字段是否允许空。可选，默认true。
    boolean nullable() default true;
    //执行insert操作的时候是否包含此字段，默认，true，可选。
    boolean insertable() default true;
    //执行update的时候是否包含此字段，默认，true，可选。
    boolean updatable() default true;
    //表示该字段在数据库中的实际类型。
    String columnDefinition() default "";
   //数据库字段的长度，可选，默认255
    int length() default 255;
}
```

（10）@Temporal 用来设置 Date 类型的属性映射到对应精度的字段。

> @Temporal(TemporalType.DATE)映射为日期 // date （只有日期）
> @Temporal(TemporalType.TIME)映射为日期 // time （是有时间）
> @Temporal(TemporalType.TIMESTAMP)映射为日期 // date time （日期+时间）

（11）@Enumerated 这个注解很好用，直接映射 enum 枚举类型的字段。
1）看源码：
```
public @interface Enumerated {
//枚举映射的类型，默认是ORDINAL（即枚举字段的下标）。
    EnumType value() default ORDINAL;
}
public enum EnumType {
    //映射枚举字段的下标
    ORDINAL,
    //映射枚举的Name
    STRING
}
```

2）看例子：
```
//有一个枚举类，用户的性别
public enum Gender {
    MAIL("男性"), FMAIL("女性");
    private String value;
    private Gender(String value) {
        this.value = value;
    }
}
//实体类@Enumerated的写法如下
@Entity
@Table(name = "tb_user")
public class User implements Serializable {
    @Enumerated(EnumType.STRING)
    @Column(name = "user_gender")
    private Gender gender;
    .......................
}
```
这时候插入两条数据，数据库里面的值是 MAIL/FMAIL，而不是“男性”/女性。

如果我们用 @Enumerated(EnumType.ORDINAL)，这时候数据库里面的值是 0，1。但是实际工作中，不建议用数字下标，因为枚举里面的属性值是会不断新增的，如果新增一个，位置变化了就惨了。

（12）@Lob 将属性映射成数据库支持的大对象类型，支持以下两种数据库类型的字段。

Clob（Character Large Ojects）类型是长字符串类型，java.sql.Clob、Character[]、char[] 和 String 将被映射为 Clob 类型。
Blob（Binary Large Objects）类型是字节类型，java.sql.Blob、Byte[]、byte[]和实现了 Serializable 接口的类型将被映射为 Blob 类型。
由于 Clob，Blob 占用内存空间较大一般配合 @Basic(fetch=FetchType.LAZY) 将其设置为延迟加载。

（13）@SqlResultSetMapping、@EntityResult、@ColumnResult 配合 @NamedNativeQuery 一起使用的。

在实际工作中不建议这样配置，因为必要性比较少，如果这种配置多了会把 @entity 用配置 XML 思路。看一个案例简单说明一下：
```
@NamedNativeQueries({
        @NamedNativeQuery(name = "getUsers",
        query = "select id,username,usertype from t_xfw_operator order by id desc",
        resultSetMapping = "usersMap")
})
@SqlResultSetMappings({
        @SqlResultSetMapping(name = "usersMap",
        entities = {},
        columns = {
                @ColumnResult(name = "id"),
                @ColumnResult(name="username"),
                @ColumnResult(name="usertype")
        })
})
@Entity
@Table(name = "operator")
public class Operator {
......
}
```


### 关联关系注解 @OneToOne、@JoinColumn、@ManyToOne、@ManyToMany、@JoinTable、@OrderBy
#### @JoinColumn 定义外键关联的字段名称
（1）源码语法
```
public @interface JoinColumn {
    //目标表的字段名,必填
    String name() default "";
    //本实体的字段名，非必填，默认是本表ID
    String referencedColumnName() default "";
    //外键字段是否唯一
    boolean unique() default false;
    //外键字段是否允许为空
    boolean nullable() default true;
    //是否跟随一起新增
    boolean insertable() default true;
    //是否跟随一起更新
    boolean updatable() default true;
}
```

（2）用法：**@JoinColumn 主要配合 @OneToOne、@ManyToOne、@OneToMany 一起使用，单独使用没有意义。**

（3）@JoinColumns 定义多个字段的关联关系。

#### @OneToOne 一对一关联关系
（1）源码语法：
```
public @interface OneToOne {
    //关系目标实体，非必填，默认该字段的类型。
    Class targetEntity() default void.class;
    //cascade 级联操作策略
  1. CascadeType.PERSIST 级联新建
  2. CascadeType.REMOVE 级联删除
  3. CascadeType.REFRESH 级联刷新
  4. CascadeType.MERGE 级联更新
  5. CascadeType.ALL 四项全选
  6. 默认，关系表不会产生任何影响
    CascadeType[] cascade() default {};
    //数据获取方式EAGER(立即加载)/LAZY(延迟加载)
    FetchType fetch() default EAGER;
    //是否允许为空
    boolean optional() default true;
    //关联关系被谁维护的。 非必填，一般不需要特别指定。   
//注意：只有关系维护方才能操作两者的关系。被维护方即使设置了维护方属性进行存储也不会更新外键关联。1）mappedBy不能与@JoinColumn或者@JoinTable同时使用。2）mappedBy的值是指另一方的实体里面属性的字段，而不是数据库字段，也不是实体的对象的名字。既是另一方配置了@JoinColumn或者@JoinTable注解的属性的字段名称。
    String mappedBy() default "";
    //是否级联删除。和CascadeType.REMOVE的效果一样。两种配置了一个就会自动级联删除
    boolean orphanRemoval() default false;
}

```

（2）用法 @OneToOne 需要配合 @JoinColumn 一起使用。注意：可以双向关联，也可以只配置一方，看实际需求。

案例：假设一个部门只有一个员工，Department 的内容如下：
```
@OneToOne
@JoinColumn(name="employee_id",referencedColumnName="id")
private Employee employeeAttribute = new Employee();
```
> 注意：employee_id指的是 Department 里面的字段，而 referencedColumnName="id" 指的是 Employee 表里面的字段。

如果需要双向关联，Employee 的内容如下：
```
@OneToOne(mappedBy="employeeAttribute")
private Department department;
```

当然了也可以不选用 mappedBy 和下面效果是一样的：
```
@OneToOne
@JoinColumn(name="id",referencedColumnName="employee_id")
private Department department;
```

#### @OneToMany 一对多 & @ManyToOne 多对一
@OneToMany & @ManyToOne 可以相对存在，也可只存在一方。
（1）@OneToMany 源码语法：
```
public @interface OneToMany {
    Class targetEntity() default void.class;
 //cascade 级联操作策略：(CascadeType.PERSIST、CascadeType.REMOVE、CascadeType.REFRESH、CascadeType.MERGE、CascadeType.ALL)
如果不填，默认关系表不会产生任何影响。
    CascadeType[] cascade() default {};
//数据获取方式EAGER(立即加载)/LAZY(延迟加载)
    FetchType fetch() default LAZY;
    //关系被谁维护，单项的。注意：只有关系维护方才能操作两者的关系。
    String mappedBy() default "";
//是否级联删除。和CascadeType.REMOVE的效果一样。两种配置了一个就会自动级联删除
    boolean orphanRemoval() default false;
}
public @interface ManyToOne {
    Class targetEntity() default void.class;
    CascadeType[] cascade() default {};
    FetchType fetch() default EAGER;
    boolean optional() default true;
}
```

@ManyToOne 与 OneToMany 的源码稍有区别仔细体会。

（2）使用案例，也是必须要和 @JoinColumn 配合使用才有效。
```
@Entity
@Table(name="user")
public class User implements Serializable{
@OneToMany(cascade=CascadeType.ALL,fetch=FetchType.LAZY,mappedBy="user")
    private Set<role> setRole; 
......}
@Entity
@Table(name="role")
public class Role {
@ManyToOne(cascade=CascadeType.ALL,fetch=FetchType.EAGER)
    @JoinColumn(name="user_id")//user_id字段作为外键
    private User user;
......}
```

#### @OrderBy 关联查询的时候的排序
一般和 @OneToMany 一起使用。

（1）源码语法：
```
@Target({METHOD, FIELD})
@Retention(RUNTIME)
public @interface OrderBy {
   /**
    * 要排序的字段，格式如下：
    *    orderby_list::= orderby_item [,orderby_item]*
    *    orderby_item::= [property_or_field_name] [ASC | DESC]
    * 字段可以是实体属性，也可以数据字段，默认ASC。
    */
    String value() default "";
}
```

（2）用法案例：
```
@Entity
@Table(name="user")
public class User implements Serializable{
@OneToMany(cascade=CascadeType.ALL,fetch=FetchType.LAZY,mappedBy="user")
@OrderBy("role_name DESC")
    private Set<role> setRole; 
......}
```

#### @JoinTable 关联关系表
@JoinTable 是指如果对象与对象之间有个关联关系表的时候，就会用到这个，一般和 @ManyToMany 一起使用。
（1）源码语法：
```
public @interface JoinTable {
    //中间关联关系表明
    String name() default "";
    //表的catalog
    String catalog() default "";
    //表的schema
    String schema() default "";
    //主链接表的字段
    JoinColumn[] joinColumns() default {};
    //被联机的表外键字段
    JoinColumn[] inverseJoinColumns() default {};
......
}
```
（2）假设 Blog 和 Tag 是多对多的关系，有个关联关系表blog_tag_relation，表中有两个属性blog_id和tag_id，那么 Blog 实体里面的写法如下：
```
@Entity
public class Blog{
    @ManyToMany
    @JoinTable(
        name="blog_tag_relation",
        joinColumns=@JoinColumn(name="blog_id",referencedColumnName="id"),
        inverseJoinColumn=@JoinColumn(name="tag_id",referencedColumnName="id")
        private List<Tag> tags = new ArrayList<Tag>();
    )
}
```

#### @ManyToMany 多对多
（1）源码语法：
```
public @interface ManyToMany {
    Class targetEntity() default void.class;
    CascadeType[] cascade() default {};
    FetchType fetch() default LAZY;
    String mappedBy() default "";
}
```
@ManyToMany 表示多对多，和 @OneToOne、@ManyToOne 一样也有单向双向之分，单项双向和注解没有关系，只看实体类之间是否相互引用。 主要注意的是当用到 @ManyToMany 的时候一定是三张表，不要想着偷懒，否则会发现有很多麻烦。

（2）案例：一个博客可以拥有多个标签，一个标签也可以使用在多个博客上，Blog 和 Tag 就是多对多关系。
```
//第一种：单向多对多
@Entity
public class Blog{
    @Id
    @Column(name = "id")
    private Integer id;
    @ManyToMany(cascade=CascadeType.ALL) 
    @JoinTable(
        name="blog_tag_relation",
joinColumns=@JoinColumn(name="blog_id",referencedColumnName="id"),      inverseJoinColumn=@JoinColumn(name="tag_id",referencedColumnName="id")
    private List<Tag> tags = new ArrayList<Tag>();
......}
//多对多中间关联关系表
@Entity
public class BlogTagRelation{
   @Column(name = "blog_id")
private Integer blogId;
   @Column(name = "tag_id")
private Integer tagId;
......}
//标签表
@Entity
public class Tag{
    @Id
    @Column(name = "id")
private Integer id;
......}
```

```
//第二种：双向多对多
@Entity
public class Blog{
    @ManyToMany(cascade=CascadeType.ALL)
    @JoinTable(
        name="blog_tag_relation",       joinColumns=@JoinColumn(name="blog_id",referencedColumnName="id"),
inverseJoinColumn=@JoinColumn(name="tag_id",referencedColumnName="id")
    private List<Tag> tags = new ArrayList<Tag>();
}
@Entity
public class Tag{
    @ManyToMany(mappedBy="BlogTagRelation")
    private List<Blog> blogs = new ArrayList<Blog>();
}
```

> 说明：BlogTagRelation 为中间关联关系表blog_tag_relation对应的实体。


### Left join、Inner join 与 @EntityGraph
#### Left join & Inner join 问题
当使用 @ManyToMany、@ManyToOne、@OneToMany、@OneToOne 关联关系的时候，当 FetchType 怎么配置 LAZY 或者 EAGER，SQL 真正执行的时候都是有一条主表查询和 N 条子表查询组成的。这种查询效率一般比较低下，子对象有多少个就会执行 N+1 条 SQL。

有时候我们需要用到 Left Join 或者 Inner Join 来查询来提高效率，只能通过 @Query 的 JQPL 语法去实现，还有后面将介绍的 Criteria API 可以做到。

**Spring Data JPA 为了简单的提高查询率，引入了 <font color=blue>EntityGraph</font> 的概念，可以解决 N+1 条 SQL 的问题。**

### @EntityGraph
JPA 2.1 推出来的 @EntityGraph、 @NamedEntityGraph 用来提高查询效率，很好的解决了 N+1 条 SQL 的问题，两者需要配合起来使用，缺一不可。@NamedEntityGraph 配置在 @Entity 上面，而 @EntityGraph 配置在 Repository 的查询方法上面，我们看一下实例。
（1）先在 Entity 里面定义 @NamedEntityGraph，其他都不变，其中 @NamedAttributeNode 可以有多个，也可以有一个。
```
@NamedEntityGraph(name = "UserInfoEntity.addressEntityList", attributeNodes = {
      @NamedAttributeNode("addressEntityList"),
      @NamedAttributeNode("userBlogEntityList")})
@Entity(name = "UserInfoEntity")
@Table(name = "user_info", schema = "test")
public class UserInfoEntity  implements Serializable {
   @Id
   @Column(name = "id", nullable = false)
   private Integer id;
   @OneToOne(optional = false)
   @JoinColumn(referencedColumnName = "id",name = "address_id",nullable = false)
   private UserReceivingAddressEntity addressEntityList;
   @OneToMany
   @JoinColumn(name = "create_user_id",referencedColumnName = "id")
   private List<UserBlogEntity> userBlogEntityList;
......
}
```
（2）只需要在查询方法上加 @EntityGraph 注解即可，其中 value 就是 @NamedEntityGraph 中的 Name，在实例配置如下：
```
public interface UserRepository extends JpaRepository<UserInfoEntity, Integer>{
   @Override
   @EntityGraph(value = "UserInfoEntity.addressEntityList")
   List<UserInfoEntity> findAll();
}
```

### 工作中关于关系查询踩过的那些坑
#### 实体里面的注解要么在 get 方法上，要么全在字段上面
所有的注解要么全配置在字段上，要么全配置在 get 方法上，不能混用，混用启动的时候就会启动不起来，然后启动的时候报错，如果是第一次使用很难找到原因，但是看语法上面这样配置又没有问题。而作者喜欢把注解都放在 get 方法上面，如下面的例子，因为 Intellij IDEA 工具生成的 Entity 类，相关的注解都在 get 方法上面，这样不需要做出任何调整，案例如下：
```
/**
 * 看一下作者实际工作中发短信的Entity如下：
 */
@Entity
@Table(name = "message_request")
@Include(rootLevel = true, type = "messageRequest")
@Setter
@ToString
public class MessageRequest extends DefaultSimpleAuditable {
    public MessageRequest(String uuid) {
        this.uuid = uuid;
    }
    public MessageRequest() {
    }
    /**
     * 下面有哪些手机号
     */
    private List<MessageRequestTelephone> messageRequestTelephone = Lists.newArrayList();
    /**
     * uuid（自动产生） 流水号
     */
    private String uuid;
    /**
     * 短信内容，不包含签名，走模板，不用这个了
     */
    @Deprecated
    private String content;
    /**
     * 哪个供应商，如果不指定，按照系统顺序发放
     */
    private String messageVendorCode;
    /**
     * 自己的模板code
     */
    private String templateCode;
    /**
     * abc,def,5模板参数，分割
     */
    private String templateParams;
    /**
     * 重试次数
     */
    private Integer retryTimes;
    /**
     * 【爱中国】，客户端传递的
     */
    private List<MessageRequestSign> messageRequestSigns;
    /**
     * 状态枚举
     */
    private MessageRequestStatus status;
    /**
     * 状态最后的异常
     */
    private String statusRemark;
    @OneToMany(cascade = CascadeType.PERSIST)
    @JoinColumn(referencedColumnName = "id", name = "message_request_id")
    @JsonManagedReference
    public List<MessageRequestSign> getMessageRequestSigns() {
        return messageRequestSigns;
    }
    @Transient
    @Exclude
    @JsonIgnore
    public String getDomesticSign() {
        if (messageRequestSigns == null) {
            return null;
        }
        List<MessageRequestSign> list = messageRequestSigns.stream().filter(s -> s.getNationCode().isDomestic()).collect(Collectors.toList());
        if (list == null || list.isEmpty()) {
            return null;
        }
        return list.stream().findFirst().get().getSign();
    }
    @Transient
    @Exclude
    @JsonIgnore
    public String getAbroadSign() {
        if (messageRequestSigns == null) {
            return null;
        }
        List<MessageRequestSign> list = messageRequestSigns.stream().filter(s -> s.getNationCode().isAbroad()).collect(Collectors.toList());
        if (list == null || list.isEmpty()) {
            return null;
        }
        return list.stream().findFirst().get().getSign();
    }
    @OneToMany(cascade = CascadeType.PERSIST)
    @JoinColumn(referencedColumnName = "id", name = "message_request_id")
    @JsonManagedReference
    public List<MessageRequestTelephone> getMessageRequestTelephone(){
        return messageRequestTelephone;
    }
    @Basic
    @Column(name = "uuid")
    public String getUuid() {
        return uuid;
    }
    @Basic
    @Column(name = "content")
    public String getContent() {
        return content;
    }
    @Basic
    @Column(name = "message_vendor_code")
    public String getMessageVendorCode() {
        return messageVendorCode;
    }
    @Basic
    @Column(name = "template_code")
    public String getTemplateCode() {
        return templateCode;
    }
    @Basic
    @Column(name = "template_params")
    public String getTemplateParams() {
        return templateParams;
    }
    //不加@Column注解，用默认的字段名字匹配规则
    public Integer getRetryTimes() {
        return retryTimes;
    }
    @Enumerated(value = EnumType.STRING)
    public MessageRequestStatus getStatus() {
        return status;
    }
    public String getStatusRemark() {
        return statusRemark;
    }
    @PostLoad
    public void postLoadDefaultValue() {
        if (status == null) {
            status = MessageRequestStatus.WAITING;
        }
    }
}
```

### 双向关联死循环问题
（1）JSON 序列化的双向关联会死循环，解决方法有两种：

- 利用 @JsonIgnore 在另外一方忽略掉（最简单，但是不推荐，这样会改变业务场景）。
- 利用 @JsonManagedReference 和 @JsonBackReference 注解来表明 back 关系，来解决 Jack Son 序列化的时候所产生的双向关联死循环。

@JsonBackReference 和 @JsonManagedReference：这两个标注通常配对使用，通常用在父子关系中。@JsonBackReference 标注的属性在序列化（serialization，即将对象转换为 JSON 数据）时，会被忽略（即结果中的 JSON 数据不包含该属性的内容）。@JsonManagedReference 标注的属性则会被序列化。在序列化时，@JsonBackReference 的作用相当于 @JsonIgnore，此时可以没有 @JsonManagedReference，但在反序列化（deserialization，即 JSON 数据转换为对象）时，如果没有 @JsonManagedReference，则不会自动注入 @JsonBackReference 标注的属性（被忽略的父或子）；如果有 @JsonManagedReference，则会自动注入自动注入 @JsonBackReference 标注的属性。

正如上面的 MessageRequest 类中描述一样，我们来开看一下 MessageRequestTelephone 的关键源码如下：
```
@Entity
@Table(name = "message_request_telephone")
@ToString(exclude = "messageRequest")
public class MessageRequestTelephone extends DefaultSimpleAuditable implements Serializable {
    /**
     * 当我们使用messageRequestId的时候，对应的@JoinColumn要注意配置insertable = false, updatable = false
     */
    private Long messageRequestId;
    @JsonBackReference
    @ManyToOne
    @JoinColumn(name = "message_request_id", referencedColumnName = "id", nullable = false, insertable = false, updatable = false)
    public MessageRequest getMessageRequest() {
        return messageRequest;
    }
......
}
```
（2）同样的当覆盖 toString() 方法的时候需要注意一下，也会有双向关联死循环的问题。解决方法一样的，toString 的时候在一方排除掉即可。


### 外键与关联关系注解的问题
（1）虽然我们使用关联查询，但是在实际工作中，所有的关联查询，表上一般是不需要建立外键约束的，为了提高操作效率，但是每个外键的字段上都会加上一般索引。

（2）不同的关联关系的配置，@JoinClumn 里面的（name、referencedColumnName）代表的意思是不一样的，很容易弄混。我们有两种方法应对：

> 打印出 SQL，我们请求的时候看一下 SQL 正确不正确，一般配置如下，key 在 spring 里面，将 SQL 参数都打印出来观察仔细。
```
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.use_sql_comments=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.type=trace
logging.level.org.hibernate.type.descriptor.sql=trace
```

> 我们现在表上建立外键约束，然后利用 Intellij IDEA 自动生成关联关系注解，当生成完代码的时候再把表上的外键约束删除掉，操作界面如下： ...

### 级联操作的使用需要注意
当我们使用关联操作的时候建议大家 cascade = CascadeType.PERSIST 配置成这个，这样 insert 的时候能帮我们不少忙，它的原理是两条都先 insert，然后再来个 update 把关联关系更新上去，读者自己打印出 SQL，也可以看得出来。如下：
```
@OneToMany(cascade = CascadeType.PERSIST)
    @JoinColumn(referencedColumnName = "id", name = "message_request_id")
    @JsonManagedReference
    public List<MessageRequestTelephone> getMessageRequestTelephone() {
        return messageRequestTelephone;
    }
	
```
级联更新、级联删除的时候比较危险，建议考虑清楚，或者完全掌握的时候再去使用，否则生产产生事故还是比较糟糕的。



















