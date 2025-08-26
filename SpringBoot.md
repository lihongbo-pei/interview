# SpringBoot

## Spring Boot 3.x

禁止循环依赖

SpringBoot为什么要禁止循环依赖?

Spring Boot（以及Spring框架）禁止循环依赖，主要是因为以下几个原因：

依赖关系复杂化： 循环依赖会导致类之间的依赖关系变得复杂和难以理解。维护一个具有循环依赖的应用程序会更加困难，可能会增加代码的耦合度，降低代码的可维护性和可读性。

实例化问题： 在Spring框架中，Bean的实例化是由Spring容器负责管理的。如果出现循环依赖，Spring容器在创建Bean时，可能会陷入死循环。例如，A依赖B，B依赖A，Spring容器在创建这两个Bean时，会一直相互等待，无法正常完成依赖注入。

构造函数注入的冲突： 当Bean的依赖通过构造函数注入时，Spring容器无法通过构造器解决循环依赖。这是因为构造函数注入要求所有依赖在创建实例时就要提供，而在循环依赖的情况下，Spring无法通过构造函数来解决问题。

解决方式： Spring框架通过setter注入（或使用其他方式，如**@Lazy**）解决了部分循环依赖问题。对于依赖注入的构造函数，如果存在循环依赖，可以通过使用代理、懒加载等机制来绕过这个问题，但这些解决方法增加了复杂性和潜在的性能开销。因此，Spring框架更倾向于避免循环依赖的出现。

```yaml
spring:
  main:
    allow-circular-references: true
```

综上，Spring Boot及Spring框架通过禁止循环依赖来保持依赖关系的简洁性、可维护性和系统的稳定性。在实际开发中，设计时应尽量避免产生循环依赖，可以通过重构代码、使用接口、事件驱动等方式来解决。



@Async

```java
@Async
@Override
public CompletableFuture<Boolean> saveOperLog(SysOperlogInfo info) {
    return CompletableFuture.completedFuture(this.save(info));
}
```



## 升级

- 在使用 `springdoc-openapi-starter-webmvc-ui` 时，可以使用 `@Tag` 和 `@Operation` 注解来替代 `@Api` 和 `@ApiOperation`

## Spring

### Spring事务什么时候会失效？

1. 方法不是 `public` 修饰的
   - Spring AOP 默认是基于**代理**机制的，**只有 `public` 方法才能被代理增强**，也就只有 `public` 方法上的 `@Transactional` 注解才会生效。
2. 方法调用是 **同类内部方法调用**（即自调用）
   - 这是最常见的事务失效问题。Spring事务依赖于 AOP 代理，如果你在一个类的方法中**直接调用本类中另一个带有 `@Transactional` 注解的方法**，事务不会生效。
3. 异常类型不匹配，事务没有回滚
   - 默认情况下，Spring 只对 **运行时异常（`RuntimeException` 及其子类）和 Error** 进行回滚。
4. 数据库不支持事务（或操作不在事务范围内）
5. 事务方法执行在线程池或异步线程中
6. 事务管理器没有配置或配置错误
7. 数据库连接自动提交（autoCommit = true）
8. 异常被捕获后没有重新抛出

### Spring IOC

#### @Autowired 和 @Resource 的区别是什么？

`Autowired` 属于 Spring 内置的注解，默认的注入方式为`byType`（根据类型进行匹配），也就是说会优先根据接口类型去匹配并注入 Bean （接口的实现类）。

**这会有什么问题呢？** 当一个接口存在多个实现类的话，`byType`这种方式就无法正确注入对象了，因为这个时候 Spring 会同时找到多个满足条件的选择，默认情况下它自己不知道选择哪一个。

这种情况下，注入方式会变为 `byName`（根据名称进行匹配），这个名称通常就是类名（首字母小写）。就比如说下面代码中的 `smsService` 就是我这里所说的名称，这样应该比较好理解了吧。

举个例子，`SmsService` 接口有两个实现类: `SmsServiceImpl1`和 `SmsServiceImpl2`，且它们都已经被 Spring 容器所管理。

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Autowired
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Autowired
private SmsService smsServiceImpl1;
// 正确注入  SmsServiceImpl1 对象对应的 bean
// smsServiceImpl1 就是我们上面所说的名称
@Autowired
@Qualifier(value = "smsServiceImpl1")
private SmsService smsService;
```

我们还是建议通过 `@Qualifier` 注解来显式指定名称而不是依赖变量的名称。

`@Resource`属于 JDK 提供的注解，默认注入方式为 `byName`。如果无法通过名称匹配到对应的 Bean 的话，注入方式会变为`byType`。

`@Resource` 有两个比较重要且日常开发常用的属性：`name`（名称）、`type`（类型）。

如果仅指定 `name` 属性则注入方式为`byName`，如果仅指定`type`属性则注入方式为`byType`，如果同时指定`name` 和`type`属性（不建议这么做）则注入方式为`byType`+`byName`。

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Resource
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Resource
private SmsService smsServiceImpl1;
// 正确注入 SmsServiceImpl1 对象对应的 bean（比较推荐这种方式）
@Resource(name = "smsServiceImpl1")
private SmsService smsService;
```

简单总结一下：

- `@Autowired` 是 Spring 提供的注解，`@Resource` 是 JDK 提供的注解。
- `Autowired` 默认的注入方式为`byType`（根据类型进行匹配），`@Resource`默认注入方式为 `byName`（根据名称进行匹配）。
- 当一个接口存在多个实现类的情况下，`@Autowired` 和`@Resource`都需要通过名称才能正确匹配到对应的 Bean。`Autowired` 可以通过 `@Qualifier` 注解来显式指定名称，`@Resource`可以通过 `name` 属性来显式指定名称。
- `@Autowired` 支持在构造函数、方法、字段和参数上使用。`@Resource` 主要用于字段和方法上的注入，不支持在构造函数或参数上使用。

### AOP

```java
@Aspect //说明这是切面
@Component // 切面也是容器中的组件

// 表示切入所有 返回类型为 ApiResult 的公共方法。
// 是一个环绕通知，会在目标方法 执行前后 执行逻辑，控制方法执行过程。
// 相比 @Before（前置通知）或 @After（后置通知），@Around 是 功能最强的通知类型
@Around("execution(public com.littlelee.base.common.util.ApiResult *(..))")
```

项目中的应用：统一拦截所有 ApiResult 返回类型的方法（Controller层的方法），若打上 @SysLog 注解，则采集操作日志信息并发送至 MQ 消息队列中，支持异常记录、耗时统计、用户信息提取等功能。

项目选用的是**AspectJ**

Spring AOP和AspectJ有什么区别？

| 特性       | Spring AOP                                                | AspectJ                                    |
| ---------- | --------------------------------------------------------- | ------------------------------------------ |
| 增强方式   | 运行时增强（基于动态代理）                                | 编译时增强、类加载时增强（直接操作字节码） |
| 切入点支持 | 方法级（Spring Bean 范围内，不支持 final 和 static 方法） | 方法级、字段、构造器、静态方法等           |
| 性能       | 运行时依赖代理，有一定开销，切面多时性能较低              | 运行时无代理开销，性能更高                 |
| 复杂性     | 简单，易用，适合大多数场景                                | 功能强大，但相对复杂                       |
| 使用场景   | Spring 应用下比较简单的 AOP 需求                          | 高性能、高复杂度的 AOP 需求                |

AOP 常见的通知类型有哪些？

![aspectj-advice-types](assets/aspectj-advice-types.jpg)

**Before**（前置通知）：目标对象的方法调用之前触发

**After** （后置通知）：目标对象的方法调用之后触发

**AfterReturning**（返回通知）：目标对象的方法调用完成，在返回结果值之后触发

**AfterThrowing**（异常通知）：目标对象的方法运行中抛出 / 触发异常后触发。AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值。

**Around** （环绕通知）：编程式控制目标对象的方法调用。环绕通知是所有通知类型中可操作范围最大的一种，因为它可以直接拿到目标对象，以及要执行的方法，所以环绕通知可以任意的在目标对象的方法调用前后搞事，甚至不调用目标对象的方法

### Spring AOP 在什么场景下会失效

一、内部方法调用失效

场景与原因

当一个 Bean 的内部方法直接调用另一个方法时，AOP 无法拦截。

原因：AOP 通过代理对象增强方法，但内部调用通过 this 直接调用目标对象的方法，绕过了代理。

解决方案：使用AopContext.currentProxy()：需开启exposeProxy配置

```java
@EnableAspectJAutoProxy(exposeProxy = true)
public class AppConfig { ... }
// 调用方式
((MyBean) AopContext.currentProxy()).methodB();
```

二、非 Spring 管理的对象

通过 new 关键字直接创建对象，而非通过 Spring 容器获取 Bean

解决方案：使用 @Component 等注解将类声明为 Bean，并通过 @Autowired 注入。

三、异步方法（@Async）

异步方法在新线程中执行时，AOP 拦截器可能无法跟踪

四、切入点表达式错误

表达式为正确匹配目标方法

```java
// 下面是项目中正确的写法
@Around("execution(public com.littlelee.base.common.util.ApiResult *(..))")
```

