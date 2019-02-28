## Auditing 与 @Version

### Auditing 及其事件详解
Auditing 翻译过来是审计和审核，Spring 的优秀之处在于帮我们想到了很多繁琐事情的解决方案，我们在实际的业务系统中，针对一张表的操作**大部分是需要记录谁什么时间创建的，谁什么时间修改的**，并且能让我们方便的记录操作日志。Spring Data JPA 为我们提供了审计功能的架构实现，提供了四个注解专门解决这件事情：

- @CreatedBy 哪个用户创建的。
- @CreatedDate 创建的时间。
- @LastModifiedBy 修改实体的用户。
- @LastModifiedDate 最后一次修改时间。

#### Auditing 如何配置
我们以一个快速的例子，看看它是怎么配置生效的。

（1）先新建一个 @Entity：UserCustomerEntity 里面的写法如下。
```
@Entity
@Table(name = "user_customer", schema = "test", catalog = "")
@EntityListeners(AuditingEntityListener.class) //2.EntityListeners
public class UserCustomerEntity {
   @Id
   @Column(name = "id", nullable = false)
@GeneratedValue(strategy = GenerationType.IDENTITY)
   private Integer id;
   @CreatedDate //1.CreatedDate
   @Column(name = "create_time", nullable = true)
   private Date createTime;
   @CreatedBy
   @Column(name = "create_user_id", nullable = true)
   private Integer createUserId;
   @LastModifiedBy
   @Column(name = "last_modified_user_id", nullable = true)
   private Integer lastModifiedUserId;
   @LastModifiedDate
   @Column(name = "last_modified_time", nullable = true)
   private Date lastModifiedTime;
   @Column(name = "customer_name", nullable = true, length = 50)
   private String customerName;
   @Column(name = "customer_email", nullable = true, length = 50)
   private String customerEmail;
......
}
```

@Entity 实体中我们需要做两点：

- 相应的字段添加 @CreatedBy、@CreatedDate、@LastModifiedBy and @LastModifiedDate注解。
- 增加 @EntityListeners(AuditingEntityListener.class)。

**（2）实现 AuditorAware 接口告诉 JPA 当前的用户是谁。**

实现 AuditorAware 接口，实现 getCurrentAuditor 方法，返回一个 Integer 的 user ID。以下代码介绍了两种做法：
```java
public class MyAuditorAware implements AuditorAware<Integer> {
   /**
    * Returns the current auditor of the application.
    * @return the current auditor
    */
   @Override
   public Integer getCurrentAuditor() {
//    第一种方式：如果我们集成了spring的Security，我们直接通过如下方法即可获得当前请求的用户ID.
//    Authentication authentication = 
      SecurityContextHolder.getContext().getAuthentication();
//    if (authentication == null || !authentication.isAuthenticated()) {
//       return null;
//    }
//    return ((LoginUserInfo) authentication.getPrincipal()).getUser().getId();
      //第二种方式通过request里面取或者session里面取
      ServletRequestAttributes servletRequestAttributes =
(ServletRequestAttributes)RequestContextHolder.getRequestAttributes();
      return (Integer) servletRequestAttributes.getRequest().getSession().getAttribute("userId");
   }
}
```
而 AuditorAware 的源码如下：
```
public interface AuditorAware<T> {
   T getCurrentAuditor();
}
```
通过实现 AuditorAware 接口的 getCurrentAuditor() 方法告诉 JPA 当前的用户是谁，里面实现方法千差万别，作者举例了两种最常见的：

- 通过 Security 取。
- 通过 Request 取。

**（3）通过 @EnableJpaAuditing 注解开启 JPA 的 Auditing 功能。**

并且告诉应用 AuditorAware 的实现类是谁，也就是我们通过 @Bean 注解把上面的实现类放到 Spring 的 Bean 管理里面，当然了也可以上面的类加上 @Component。具体配置方式如下：
```
@SpringBootApplication
@EnableJpaAuditing
public class QuickStartApplication {
   public static void main(String[] args) {
      SpringApplication.run(QuickStartApplication.class, args);
   }
   @Bean
   public AuditorAware<Integer> auditorProvider() {
      return new MyAuditorAwareImpl();
   }
}
```
验证结果如下。

通过以上的三步，我们已经完成了 auting 的配置，通过 userCustomerRepository.save(new UserCustomerEntity("1","Jack")); 的执行，我们看数据库里面的 4 个字段已经给填上去了。

@MappedSuperclass
实际工作中我们还会对上面的实体部分进行改进，引入 @MappedSuperclass 注解，我们将 @Id、@CreatedBy、@CreatedDate、@LastModifiedBy and @LastModifiedDate 抽象到一个公用的基类里面，方便公用和形成每个表的字段约束。可以将其放到我们公司的框架代码上，对表设计形成统一的强约束。

步骤如下：

（1）改进后我们新增一个 AbstractAuditable 的抽象类：
```
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class AbstractAuditable {
   @Id
   @Column(name = "id", nullable = false)
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   private Integer id;
   @CreatedDate
   @Column(name = "create_time", nullable = true)
   private Date createTime;
   @CreatedBy
   @Column(name = "create_user_id", nullable = true)
   private Integer createUserId;
   @LastModifiedBy
   @Column(name = "last_modified_user_id", nullable = true)
   private Integer lastModifiedUserId;
   @LastModifiedDate
   @Column(name = "last_modified_time", nullable = true)
   private Date lastModifiedTime;
......
}
```

（2）而我们每个需要 Auditing 的实体只需要继承 AbstractAuditable 即可。

内容如下：
```
@Entity
@Table(name = "user_customer", schema = "test", catalog = "")
public class UserCustomerEntity extends AbstractAuditable {
   @Column(name = "customer_name", nullable = true, length = 50)
   private String customerName;
   @Column(name = "customer_email", nullable = true, length = 50)
   private String customerEmail;
......}
```

### Auditing 原理解析
（1）我们先看一下关键的几个源码的关系图：

![](http://images.gitbook.cn/8cfa5800-4186-11e8-94d7-4b37be7eacf0)

```
@Configurable
public class AuditingEntityListener {
   private ObjectFactory<AuditingHandler> handler;
   public void setAuditingHandler(ObjectFactory<AuditingHandler> auditingHandler) {
      Assert.notNull(auditingHandler, "AuditingHandler must not be null!");
      this.handler = auditingHandler;
   }
   //在新增之前通过handler来往我们的@Entity里面的auditor的那些字段塞值。
   @PrePersist
   public void touchForCreate(Object target) {
      if (handler != null) {
         handler.getObject().markCreated(target);
      }
   }
   //在更新之前通过handler来往我们的@Entity里面的auditor的那些字段塞值。
   @PreUpdate
   public void touchForUpdate(Object target) {
      if (handler != null) {
         handler.getObject().markModified(target);
      }
   }
}
```

（3）通过调用关系图和 AuditingEntityListener，我们其实可以发现以下两点情况：

AuditingEntityListener 通过委托设计模式，委托 AuditingHandler 进行处理，而我们看 AuditingHandler 的源码会发现，里面就是根据 ID 和 Version（后面介绍）来判断我们的对象是新增还是更新，从而来更改时间字段和 User 字段。而 User 字段是通过 AuditorAware 的实现类来取的，并且 AuditorAware 没有默认实现类，只有我们自己的实现类，也就是 AuditorAware 的实现类必须我们自己来定义，否则启动会报错。
AuditingEntityListener 的代码如此简单，我们能不能自定义呢？答案是肯定的，通过 @PrePersist、@PreUpdate 查看源码得出，Java Persistence API 底层又帮我们提供的 Callbacks，而这些回调方法，用于侦听保存、查询（抓取）、更新和删除数据库中的数据。

### Listener 事件的扩展
#### 自定义 EntityListener
随着 DDD 的设计模式逐渐被大家认可和热捧，JPA 通过这种 Listener 这种机制可以很好的实现事件分离、状体分离。假如，订单的状态变化可能对我们来说比较重要，我们需要定一个类去监听订单状态变更，通知相应的逻辑代码各自去干各自的活。

**（1）新增一个 OrderStatusAuditListener 类，在相应的操作上添加 Callbacks 注解。
```
public class OrderStatusAuditListener {
   @PostPersist
   private void postPersist(OrderEntiy entity) {
      //当更新的时候做一些逻辑判断，及其事件通知。
   }
   @PostRemove
   private void PostRemove(OrderEntiy entity) {
      //当删除的时候做一些逻辑判断。
   }
   @PostUpdate 
   private void PostUpdate(OrderEntiy entity) {
      //当更新的时候
      // entity.getOrderStatus()，做一些逻辑判断
   }
}
```

**（2）我们的订单实体变化如下：**
```
@Entity
@Table("orders")
@EntityListeners({AuditingEntityListener.class, OrderStatusAuditListener.class})
public class OrderEntity  extends AbstractAuditable{
   @Enumerated(EnumType.STRING)
   @Column("order_status")
   private OrderStatusEnum orderStatus;
  ...... 
}

```

即可完成自定义 EntityListener。

**实际工作记录操作日志的实例**
```
public class ActionsLogsAuditListener {
   private static final Logger logger = LoggerFactory.getLogger(ActionsLogsAuditListener.class);
   @PostLoad
   private void postLoad(Object entity) {
      this.notice(entity, OperateType.load);
   }
   @PostPersist
   private void postPersist(Object entity) {
      this.notice(entity, OperateType.create);
   }
   @PostRemove
   private void PostRemove(Object entity) {
      this.notice(entity, OperateType.remove);
   }
   @PostUpdate
   private void PostUpdate(Object entity) {
      this.notice(entity, OperateType.update);
   }
   private void notice(Object entity, OperateType type) {
      logger.info("{} 执行了 {} 操作", entity, type.getDescription());
      //我们通过active mq 异步发出消息处理事件
      ActiveMqEventManager.notice(new ActiveMqEvent(type, entity));
   }
   enum OperateType {
      create("创建"), remove("删除"),update("修改"),load("查询");
      private final String description;
      OperateType(String description) {
         this.description=description;
      }
      public String getDescription() {
         return description;
      }
   }
}
```
我们通过自定义的 ActionsLogsAuditListener 来监听我们要处理日志的实体，然后将事件变更，<font color=blue>通过消息队列进行异步处理，这样就可以完全解耦了。当然了，这里我们解耦的方式也可以通过 Spring 的事件机制进行解决</font>。通过工作中的此示例，来帮助大家更好的理解 Audit 的机制，顺便说一下处理操作的日志的正确思路，记录当前真实发生的数据和状态，及其时间即可，具体变化了什么那是在业务展示层面上要做的事情，这里没有必要做比对的事情，记住这一点之后就会让你的日志处理实现机制豁然明朗，变得容易许多。


### @Version 处理乐观锁的问题
#### @Version 乐观锁介绍
我们在研究 Auditing 的时候，发现了一个有趣的注解 @Version，源码如下：
```
package org.springframework.data.annotation;
/**
 * Demarcates a property to be used as version field to implement optimistic locking on entities.
 */
@Retention(RUNTIME)
@Target(value = { FIELD, METHOD, ANNOTATION_TYPE })
public @interface Version {}

```

对于数据来说，简单理解：在数据库并发操作时，为了保证数据的正确性，我们会做一些并发处理，主要就是加锁。在加锁的选择上，常见有两种方式：悲观锁和乐观锁。

- 悲观锁：简单的理解就是把需要的数据全部加锁，在事务提交之前，这些数据全部不可读取和修改。
- 乐观锁：使用对单条数据进行版本校验和比较，来对保证本次的更新是最新的，否则就失败，效率要高很多。在实际工作中，乐观锁不止在数据库层面，其实我们在做分布式系统的时候，为了实现分布式系统的数据一致性，分布式事物的一种做法就是乐观锁。


#### 数据库操作举例说明
**悲观锁的做法：**
```
select * from user where id=1 for update;
update user  set name='jack'  where id=1;

```
通过使用 for update 给这条语句加锁，如果事务没有提交，其他任何读取和修改，都得排队等待。在代码中，我们加事务的 Java 方法就会自然的形成了一个锁。

**乐观锁的做法：**
```
select uid,name,version from user where id=1;
update user set name='jack', version=version+1 where id=1 and version=1
```
假设本次查询 version=1，在更新操作时，带上这次查出来的 Version，这样只有和我们上次版本一样的时候才会更新，就不会出现互相覆盖的问题，保证了数据的原子性。

#### @Version 用法
在没有 @Version 之前，我们都是自己手动维护这个 Version 的，这样很有可能做什么操作的时候给忘掉。或者是我们自己底层做框架，用 AOP 的思路做拦截底层维护这个 Version 的值。而 Spring Data JPA 的 @Version 就是通过 AOP 机制，帮我们动态维护这个 Version，从而更优雅的实现乐观锁。

**（1）实体上的 Version 字段加上 @Version 注解即可。**

我们对上面的实体 UserCustomerEntity 改进如下：
```
@Entity
@Table(name = "user_customer", schema = "test", catalog = "")
public class UserCustomerEntity extends AbstractAuditable {
   //新增控制乐观锁的字段。并且加上@Version注解
   @Version
   @Column(name = "version", nullable = true)
   private Long version;
......
}
```

（2）实际调用
```
userCustomerRepository.save(new UserCustomerEntity("1","Jack"));
UserCustomerEntity uc= userCustomerRepository.findOne(1);
uc.setCustomerName("Jack.Zhang");
userCustomerRepository.save(uc);
```
我们会发现 Insert 和 Update 的 SQL 语句都会带上 Version 的操作。当乐观锁更新失败的时候，会抛出异常 org.springframework.orm.ObjectOptimisticLockingFailureException。

实现原理关键代码
（1）SimpleJpaRepository.class 里面的 save 方法如下：
```
public <S extends T> S save(S entity) {
   if (entityInformation.isNew(entity)) {
      em.persist(entity);
      return entity;
   } else {
      return em.merge(entity);
   }
}
```
（2）如果我们在此处设置一个 debug 断点的话，我们一步一步往下面走会发现进入 JpaMetamodelEntityInformation.class 的关键代码如下：
```
    @Override
    public boolean isNew(T entity) {
        if (!versionAttribute.isPresent()
                || versionAttribute.map(Attribute::getJavaType).map(Class::isPrimitive).orElse(false)) {
            return super.isNew(entity);
        }
        BeanWrapper wrapper = new DirectFieldAccessFallbackBeanWrapper(entity);
        return versionAttribute.map(it -> wrapper.getPropertyValue(it.getName()) == null).orElse(true);
    }
```
所以到这里，可以看出当我们更新的时候，若实体对象上面有 @Version 注解，那么就一定要带上 version，如果没带上 version 字段的值，只有 ID 字段的值，系统也会认为是新增。相反，如果我们没有 @Version 注解的字段，那么就会以 @ID 字段来判断是否是新增。其实这里我们也明白，省去了传统都需要我们自己去实现的 saveOrUpdate 方法。


（3）其实我们多看看代码，多 debug 几次就会发现，也可以在 @Entity 的类里面覆盖掉 isNew() 方法，这样可以实现自己的 isNew 的判断逻辑。
```
@Entity
@Table(name = "user")
public class UserEntity  implements Persistable {
    @Transient   //这个注解表明这个字段不是持久化的
    @JsonIgnore //json显示的时候我们也可忽略这个字段
    @Override
    public boolean isNew() {
        return getId() == null;
    }
    ....
}
```


































































































































































