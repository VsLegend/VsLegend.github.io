# 声明式事务注解:@Transactional

## Spring中的事务配置
Spring配置文件中关于事务配置由三个组成部分，分别是DataSource、TransactionManager和代理机制这三部分，无论哪种配置方式，一般变化的只是代理机制这部分。

- 配置数据源：把一个DataSource作为一个@Bean注册到Spring容器中。
```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/springmvc?useSSL=false&amp;serverTimezone=Hongkong&amp;characterEncoding=utf-8&amp;autoReconnect=true"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>
```

- 启用Spring事务管理器：在 xml 配置文件中添加事务配置信息。除了用配置文件的方式，@EnableTransactionManagement注解也可以启用事务管理功能。
```xml
<--Spring xml中的事务管理器配置--!>
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<--这里引用的是数据源的bean--!>
    <property name="dataSource" ref="dataSource" />
</bean> 
```

- 使用事务功能：将@Transactional 注解添加到方法或类上，并设置相关属性。


<br><br>

---


## 属性及其作用
> @Transactional 注解也可以添加到类级别上，此时该注解默认适用于该类以及该类所有子类的公共方法上。表示所有该类的公共方法都配置相同的事务属性信息。
>
> 方法级别的事务属性信息会覆盖类级别的相关配置信息。


```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {

	@AliasFor("value")
	String transactionManager() default "";

	Propagation propagation() default Propagation.REQUIRED;

	Isolation isolation() default Isolation.DEFAULT;

	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

	boolean readOnly() default false;

	Class<? extends Throwable>[] rollbackFor() default {};

	String[] rollbackForClassName() default {};

	Class<? extends Throwable>[] noRollbackFor() default {};

	String[] noRollbackForClassName() default {};

}
```

**作用范围**

可以将注解放在接口、类的定义上，也可以直接放在方法上。它们根据优先级顺序相互覆盖，优先级从最低到最高为：接口、超类、类、接口方法、超类方法和类方法。

@Transactional用在类级别上时，将作用在此类的所有公共方法；放在私有或受保护的方法上，将会被忽略。



**属性作用**

⭐ name	当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器，为空时取值为。

⭐ propagation	事务的传播行为。定义了逻辑业务的事务边界，默认值为 REQUIRED。

⭐ isolation	事务的隔离度。描述了并发事务下各事务对彼此的能见程度，每种隔离级别可以防止对事务产生的零个或多个并发副作用，包括胀读、不可重复读、幻读。默认值采用 DEFAULT。

⭐ timeout	事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。

⭐ read-only	指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true，个人认为提供这种只读的事务查询机制是为了使用mybatis的本地缓存？

⭐ rollback-for	用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。

⭐ no-rollback- for	抛出 no-rollback-for 指定的异常类型时，不回滚事务。

<br><br>

 
### 事务传播机制-propagation
**物理事务与逻辑事务**：

要搞懂传播机制，那么就要明白逻辑事务中各个事务的关系，如此才能理解透彻。

事务性资源实际打开的事务就是物理事务，如数据库事务。

而Spring会为每个@Transactional注解的方法创建一个事务范围，这种事务无法影响数据库事务的实现，因此可以理解为逻辑事务。

在逻辑事务中，各个事务的关系可以是并列、覆盖或包含。

> 对于并排：并发情况下，多个线程相互执行自己的事务，互不影响（单指事务异常回滚，不涉及事务隔离）。
> 
> 对于覆盖：是指层层嵌套的方法共用同一个事务，事务的属性由最外围方法的属性值决定。这些方法要么同时执行成功要么同时回滚。
> 
> 对于包含：大范围的事务称为外围事务，小范围的事务称为内部事务，外围事务可以包含内部事务。他们在逻辑上是互相独立的。每个内部事务，都能独立设置如read-only等属性，而不影响外围事务。


如何处理逻辑事务内部以及与物理事务之间的关联关系，就是传播特性解决的问题。


<br>


**事务的传播类型**：

1. REQUIRED：Spring默认的事务传播机制。支持当前事务；如果不存在事务，则创建一个新的事务。


> 参与到一个已存在的更大范围的外围事务中，如果没有外围事务，就打开一个新事务用于当前范围。在相同的线程中，这是一种很好的事务安排方式。
> 
> 注：一个参与到外围事务的事务，会使用外围事务的特性，安静地忽略掉自己的隔离级别，超时值，只读标识等设置。
>
> 当然可以在事务管理器上设置validateExistingTransactions标识为true，这样当你自己的事务和参与到的外围事务设置不一样时会被拒绝。

2. REQUIRES_NEW：创建一个新事务，如果当前存在事务，则暂停当前事务。

> 与REQUIRED相比，总是使用一个独立的事务用于当前的逻辑事务，从来不参与到一个已存在的外围事务范围。
>
> 这样安排的话，底层的事务资源是不同的，因此，可以独立地提交或回滚。外围事务不会被内部事务的回滚状态影响。
>
> 这样一个独立的内部事务可以声明自己的隔离级别，超时时间和只读设置，并不继承外围事务的特性，也不会被外围事务覆盖。
>
> 与NOT_SUPPORTED类似，我们需要JTATransactionManager来实现实际的事务暂停。

3. MANDATORY（强制的）：仅支持当前事务；如果当前事务不存在，则抛出异常。

4. NESTED（嵌套）：如果存在当前事务，则在该事务种嵌套执行。同时将在这段逻辑事务开始处标记一个保存点，当这个逻辑事务抛出异常回滚时，那么事务将跳转到保存点，从而不影响嵌套事务的外部。
当前事务不存在时，同REQUIRED。

> 当事务存在时，使用同一个事务，该事务可能会带有多个保存点，回滚到这些保存点从而不会影响整个事务。可以认为是部分回滚，这样一个内部事务范围触发了一个回滚，外围事务能够继续这个物理事务，尽管有一些操作已经被回滚。
>
> DataSourceTransactionManager支持这种开箱即用的传播。
>
> JTATransactionManager的一些实现也可能支持这一点。不过JpaTransactionManager仅支持JDBC连接的NESTED，
> 如果我们将nestedTransactionAllowed标志设置为true，且JDBC驱动程序支持保存点，它也适用于JPA事务中的JDBC访问代码。


5. NEVER：非事务执行，如果存在事务，则抛出异常。

6. SUPPORTS：支持当前事务，如果不存在则以非事务方式执行。

7. NOT_SUPPORTED：以非事务方式执行，如果当前事务存在，则挂起当前事务。

> JTATransactionManager开箱即用地支持真正的事务暂停。其他人通过持有对现有的引用然后从线程上下文中清除它来模拟暂停

8. TIMEOUT_DEFAULT：使用默认超时的底层事务系统，或者如果不支持超时则没有。

<br><br>

### 事务的隔离级别-isolation
如下隔离级别跟数据库隔离级别相同。并发情况下，写读是脏读，读写读是不可重复读，where insert where是幻读。
> 脏读：读取一个并发事务的未提交更改
>
> 不可重复读取：如果并发事务更新同一行并提交，则在重新读取一行时获得不同的值
>
> 幻读：如果另一个事务添加或删除范围中的某些行并提交，则在重新执行范围查询后获取不同的行

<br>

1. DEFAULT：这是默认的隔离级别。跟随底层数据库的隔离规则。MySQL的默认隔离级别是REPEATABLE-READ，即可重复读。


2. READ_UNCOMMITTED：读未提交。可能发生胀读、不可重复读和幻读。是最低的隔离级别，允许最多的并发访问。

> Postgres 不支持READ_UNCOMMITTED隔离，而是回退到READ_COMMITED。此外，Oracle 不支持或不允许READ_UNCOMMITTED。


3. READ_COMMITTED：已提交读。能够阻止胀读；可能发生不可重复读和幻读。

4. REPEATABLE_READ：可重复读，能够阻止胀读和不可重复读；可能发生幻读。

> 它是防止更新丢失所需的最低级别。当两个或更多并发事务读取和更新同一行时，会发生更新丢失。REPEATABLE_READ根本不允许同时访问一行。因此，丢失的更新不会发生。
>
> REPEATABLE_READ是 Mysql 中的默认级别。Oracle 不支持REPEATABLE_READ。

5. SERIALIZABLE：可串行化，胀读、不可重复读和虚读都不会发生。它可以防止所有提到的并发副作用，但并发访问量低。它会产生锁竞争，造成性能压力以及大量超时。



<br><br>




### 事务回滚-rollback
rollback参数用于指定抛出何种异常时才进行回滚，适用于该异常类的子类。默认配置下 Spring 只会回滚RuntimeException异常或Error。

> **checked异常和unchecked异常**：
>
> checked异常继承Exception，主要是程序外部引起的异常，如无效用户输入，文件不存在，网络IOException或数据库链接SQLException错误。
>
> unchecked异常继承RuntimeException（继承Exception），主要是逻辑异常，IllegalArgumentException, NullPointerException和IllegalStateException等异常。



<br><br>

---

## 使用SpringAOP的自调用问题
在Spring的AOP代理下，只有目标方法由外部调用，目标方法才由 Spring 生成的代理对象来管理。当目标方法由内部调用时则不会，这时就会产生自调用问题。

如下，不带注解的方法调用带注解的方法时，有@Transactional注解的方法其事务将被忽略，出现异常时也不会发生回滚。
```java
@Service
public class OrderService {
    public void insert() {
       insertOrder();
    }
    @Transactional
    public void insertOrder() {
        // doSomething
    }
}
```

@Transactional自调用问题是由于使用Spring AOP代理造成的。为解决这两个问题，可以使用 AspectJ 取代Spring AOP代理。



<br><br>

---



## 事务管理器的个性化使用方法
当你发现@Transactional在不同方法上使用了相同属性，可以使用自定义注解来简化事务的使用。

例如：有多个事务管理器，且每个事务管理器都被多个方法使用，就可以通过如下方法实现注解的使用。
```xml
<bean id="transactionManager1" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    ...
    <qualifier value="order"/>
</bean>

<bean id="transactionManager2" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    ...
    <qualifier value="account"/>
</bean>
```

自定义注解
```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional("order")
public @interface OrderTx {
}

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional("account")
public @interface AccountTx {
}
```

注解的使用
```java
public class TransactionalService {

    @OrderTx
    public void setSomething(String name) { ... }

    @AccountTx
    public void doSomething() { ... }
}
```


<br><br>

---


## 参考：

[Transaction Management](https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/transaction.html)

[Annotation Type Transactional](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html)

[Enum Propagation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Propagation.html#MANDATORY)

[Spring 事务 -- @Transactional的使用](https://www.jianshu.com/p/befc2d73e487)

[Transaction Propagation and Isolation in Spring @Transactional](https://www.baeldung.com/spring-transactional-propagation-isolation)

[Spring Boot 指定事务管理器](https://www.appblog.cn/2020/02/22/Spring%20Boot%20%E6%8C%87%E5%AE%9A%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86%E5%99%A8/)

[@Transactional 指定 rollbackFor](https://github.com/alibaba/p3c/issues/799)
