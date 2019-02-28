## 对 MVCWeb 的支持分页和排序的支持
在实际工作中，我们经常有排序和分页的需求，很多小伙伴都在写自己的 Page 对象和排序逻辑，通过本节内容我们来看下 Spring Data JPA 对分页和排序做了哪些支持。

### 配置方法
#### 利用 @EnableSpringDataWebSupport
Spring Data 附带各种 Web 支持如果模块支持库的编程模型。通过 @EnableSpringDataWebSupport 这个注解可以启用 Web 集成支持。@EnableSpringDataWebSupport 注解配置在 JavaConfig 类上即可，如下：
```
@Configuration
@EnableWebMvc
//开启支持Spring Data web的支持
@EnableSpringDataWebSupport
public class WebConfiguration { }
```
@Controller 上直接使用 org.springframework.data.domain.Pageable 接收 Page 和分页相关参数，利用 org.springframework.data.domain.Page 可以返回相关的 Page 对象的值，如下：

```
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;
@Controller
@RequestMapping(path = "/demo")
public class UserInfoController {
   @Autowired
   private UserRepository userRepository;
/**
 * 案例1：使用分页和排序的 Pageable 对象返回 Page 对象。
 * @param pageable
 * @return
 */
@RequestMapping(path = "/user/page")
@ResponseBody
public Page<UserInfoEntity> findAllByPage(Pageable pageable) {
   return userRepository.findAll(pageable);
}
/**
 * 案例2：单独使用排序，返回 HttpEntity 结果
 * @param sort
 * @return
 */
@RequestMapping(path = "/user/sort")
@ResponseBody
public HttpEntity<List<UserInfoEntity>> findAllBySort(Sort sort) {
   return new HttpEntity(userRepository.findAll(sort));
}
}
```


|Pageable 里面的字段	|描述|
|---|---|
|page	|你想要查找的第几页，如果你不传，默认是 0|
|size	|分页大小，默认是 20|
|sort	|属性，应按格式 property,property(ASC或DESC)。默认排序升序从小到大 ASC，使用多个 sort 参数，如果你想切换方向，例如，?sort=firstname&sort=lastname,asc|


所以请求的方式如下。

（1）$ curl http://127.0.0.1:8080/demo/user/page
```
{
  "content": [
    //UserInfoEntity的20条数据
  ],
  "last": false,
  "totalPages": 3,
  "totalElements": 41,
  "size": 20,
  "number": 0,
  "sort": null,
  "first": true,
  "numberOfElements": 20
}
```
我们看到返回结果有两部分组成：

一是 content，即返回的内容结果。
二是 page 本身的一些信息。

（2）$ curl http://127.0.0.1:8080/demo/user/page?page=2&size=5
```
{
  "content": [
//第二页的 UserInfoEntity 的5条数据
  ],
  "last": false,
  "totalPages": 9,
  "totalElements": 41,
  "size": 5,
  "number": 2,
  "sort": null,
  "first": false,
  "numberOfElements": 5
}
```
我们看到返回结果分页的页数变了，这种结构使得我们的 API 接口相当的灵活，可以仔细体会一下。

（3）$curl http://127.0.0.1:8080/demo/user/page?page=2&size=5&sort=firstName
```
{
  "content": [
//第二页的UserInfoEntity的5条数据
  ],
  "last": false,
  "totalPages": 9,
  "totalElements": 41,
  "size": 5,
  "number": 2,
  "sort": [{"direction":"ASC","property":"firstName","ignoreCase":false,"nullHandling":"NATIVE","ascending":true,"descending":false}],
  "first": false,
  "numberOfElements": 5
}
```
（4）$curl http://127.0.0.1:8080/demo/user/sort?sort=firstName,desc按照名称倒序显示结果。

### 原理分析
当我们配置了 @EnableSpringDataWebSupport 的注解之后，Spring 容器将会帮我们配置设置将注册几个基本组成部分：

- 一个 DomainClassConverter 让 Spring MVC 解决的实例库管理域类来自请求参数或路径变量。
- HandlerMethodArgumentResolver 实现让 Spring MVC 解决可分页和排序实例来自请求参数。

#### DomainClassConverter 组件
DomainClassConverter 允许使用域类型在你的 Spring MVC 控制器直接方法签名，这句话怎么理解呢？看下面的实例：
```
@Controller
@RequestMapping("/user")
public class UserController {
  @RequestMapping("/{id}")
  public UserInfoEntity getUserInfo(@PathVariable("id") UserInfoEntity userInfoEntity) {
    return user;
  }
}
```
我们看到 Controller 里面没有引用任何 userRepository，但是，我们测试这个请求的时候，user 里面是有实体的数据库里面的值。@EnableSpringDataWebSupport 这个注解注入的 DomainClassConverter 组件，帮我们解决通过了让 Spring MVC path **变量转换成的 ID 类型域类，最终通过调用访问实例**，达到了 userRepository.findOne(id) 的效果。

#### HandlerMethodArgumentResolvers 可分页和排序
学习过 Spring MVC 的同学都知道实现 HandlerMethodArgumentResolver 接口可以自定义参数解析。而 Spring Data JPA 正是利用此特性，有两个参数解析类：PageableHandlerMethodArgumentResolver 的实例和 SortHandlerMethodArgumentResolver 的实例，帮我们解析 URL 里面的 Query Param 的 Page 相关的和 Sort 相关的参数。
**（1）@EnableSpringDataWebSupport 注解帮我们导入 SpringDataWebConfiguration 关键源码如下：**
```
public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        List<String> imports = new ArrayList<>();
    imports.add(ProjectingArgumentResolverRegistrar.class.getName());
            imports.add(resourceLoader//
                    .filter(it -> ClassUtils.isPresent("org.springframework.hateoas.Link", it))//
                    .map(it -> HateoasAwareSpringDataWebConfiguration.class.getName())//
                    .orElseGet(() -> SpringDataWebConfiguration.class.getName()));
            resourceLoader//
                    .filter(it -> ClassUtils.isPresent("com.fasterxml.jackson.databind.ObjectMapper", it))//
                    .map(it -> SpringFactoriesLoader.loadFactoryNames(SpringDataJacksonModules.class, it))//
                    .ifPresent(it -> imports.addAll(it));
            return imports.toArray(new String[imports.size()]);
        }
```
（2）SpringDataWebConfiguration 帮我们加载 SortHandlerMethodArgumentResolver 和 PageableHandlerMethodArgumentResolver
（3）PageableHandlerMethodArgumentResolver 的关键源码如下：

通过此段源码其实也可以发现 Spring Data JPA 有默认分页的大小，最大 2000 size，主要解析 page 和 size 参数。
```
public class PageableHandlerMethodArgumentResolver implements PageableArgumentResolver {
    private static final SortHandlerMethodArgumentResolver DEFAULT_SORT_RESOLVER = new SortHandlerMethodArgumentResolver();
    private static final String INVALID_DEFAULT_PAGE_SIZE = "Invalid default page size configured for method %s! Must not be less than one!";
    private static final String DEFAULT_PAGE_PARAMETER = "page";
    private static final String DEFAULT_SIZE_PARAMETER = "size";
    private static final String DEFAULT_PREFIX = "";
    private static final String DEFAULT_QUALIFIER_DELIMITER = "_";
    private static final int DEFAULT_MAX_PAGE_SIZE = 2000;
    static final Pageable DEFAULT_PAGE_REQUEST = PageRequest.of(0, 20);
    @Override
    public Pageable resolveArgument(MethodParameter methodParameter, @Nullable ModelAndViewContainer mavContainer,
            NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) {
        assertPageableUniqueness(methodParameter);
        Optional<Pageable> defaultOrFallback = getDefaultFromAnnotationOrFallback(methodParameter).toOptional();
        String pageString = webRequest.getParameter(getParameterNameToUse(pageParameterName, methodParameter));
        String pageSizeString = webRequest.getParameter(getParameterNameToUse(sizeParameterName, methodParameter));
......
return PageRequest.of(p, ps,
                sort.isSorted() ? sort : defaultOrFallback.map(Pageable::getSort).orElseGet(Sort::unsorted));
}}
```

通过此段源码其实还可以发现 PageRequest 是 Pageable 的默认实现类，此处给我们提供了一种思路，当使用 RPC 的 Service 的调用的时候，可以用过 new PageRequest 传递分页逻辑。

#### Page 返回结果的关键部分
我们通过 SimpleJpaRepository 的部分源码可以发现：PageImpl 是 Page 的返回结果的实现类，如下：
```
public class SimpleJpaRepository
    public Page<T> findAll(Pageable pageable) {
        if (isUnpaged(pageable)) {
            return new PageImpl<T>(findAll());
        }
        return findAll((Specification<T>) null, pageable);
    }
```

#### @PageableDefault 改变默认的 Page 和 size
我们假设默认显示第三页的内容，默认一个的大小是10条：
```
@RequestMapping(path = "/user/page")
@ResponseBody
public Page<UserInfoEntity> findAllByPage(@PageableDefault(page = 3,size = 10) Pageable pageable) {
   return userRepository.findAll(pageable);
}
```

Dubbo 等 RPC 的使用建议：

在实际工作中，由于微服务的整个环境，我们可能通过 RPC 协议，如 Dubbo 等对外提供 Service 的服务，Service 的接口的 jar 要尽量的少引用和接口本身无关的 jar，所以我们发现，其实上面说的这些对 MVC 的 Page 的支持，都是在 Spring Data Common 的 jar 里面，所以只要对外多引用这一个包即可。








