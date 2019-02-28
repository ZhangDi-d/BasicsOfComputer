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



















































































































































































