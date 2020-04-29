---
layout:     post
title:      "Spring的事务管理"
subtitle:   "Spring事务管理与事务回滚"
date:       2020-04-30 00:11:54
author:     "silhouette"
header-img: "img/images/spring/05/00-post-bg.jpg"
tags:
    - Spring
---

## 前言

一直对Spring的事务管理理解不够透测，并且开发时老是会出现报错后事务未回滚的情况，此次正好有时间整理一下，好好的把Spring的事务管理的实现与事务回滚弄清楚。以下内容设计到的参考资料有：

* [Spring事务管理之几种方式实现事务](https://blog.csdn.net/chinacr07/article/details/78817449)
* [Spring中@Transactional用法深度分析之一](https://blog.csdn.net/blueheart20/article/details/44654007?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-4&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-4)
* [@Transactional](https://blog.csdn.net/mingyundezuoan/article/details/79017659)
* [spring事物配置，声明式事务管理和基于@Transactional注解的使用](https://blog.csdn.net/bao19901210/article/details/41724355?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1)
* [Spring事务管理（详解+实例）](https://blog.csdn.net/trigl/article/details/50968079)
* [spring 事务回滚](https://www.cnblogs.com/0201zcr/p/5962578.html)
* [浅谈Spring中的事务回滚](https://www.cnblogs.com/zeng1994/p/8257763.html)

## 一、Spring 中的事务管理

#### 1.1 事务认识

* `事务(Transaction)`：它是一些列严密操作动作。要么都操作完成，要么都会滚撤销。Spring事务管理基于底层数据库本身的事务处理机制。事务具备`ACID`四种特性，ACID 是Atomic(原子性)、Consistency(一致性)、Isolation(隔离性)和Durability(持久性)的英文缩写。
  1. 原子性(Atomicity)
     * 事务最基本的操作单元，要么全部成功，要么全部失败，不会结束在中间某个环节。事务在执行过程中发生错误，会被会滚到事务开始前的状态，就像这个事务从来没有执行过一样。
  2. 一致性(Consistency)
     * 事务的一致性指的是在一个事物执行之前和执行之后数据库都必须处于一致性状态。如果事务成功地完成，那么系统中所有变化将正常地应用，系统处于有效状态。如果在事务中出现错误，那么系统中的所有变化将自动会滚。
  3. 隔离性
     * 事务的隔离性指的是在兵法环境中，当不同的事物同时操纵相同的数据时，每个事物都有各自的完整数据空间。由并发事务所做的修改必须与任何其它并发事务所做的修改隔离。事务查看数据更新时，数据所处的状态要么是另一事务修改它之前的状态，要么是另一事务修改它之后的状态，事务不会查看到中间状态的数据。
  4. 持久性
     * 事务的持久性指的是只要事务成功结束，它对数据库所做的更新必须永久保存下来。即使发生系统崩溃，重新启动数据库系统后，数据库还能恢复到事务成功结束时的状态。

#### 1.2 明确Spring 事务控制

1. JavaEE 体系进行分层开发，事务处理位于业务层，Spring 提供了分层设计`业务层`的事务处理解决方案
2. Spring 框架为我们提供了一组事务控制的接口。这组接口是在 spring-tx.xxx.jar中。
3. Spring 的事务控制都是基于 AOP 的，它即可以使用编程的方式实现，也可以使用配置的方式实现。

#### 1.3 Spring 中事务控制的 API介绍

`Spring所有的事务管理策略类都继承自 org.springframework.transaction.platformTransactionManager接口`

<img src="/img/images/spring/05/01-spring-interface.png" alt="01-spring-interface" />

* `PlatformTransactionManager`：此接口上 Spring 的事务管理器，它里面提供了我们常用的操作事务的方法

  * `TransactionStatus getTransaction(TransactionDefinition definition)`：获取事务状态信息
  * `void commit(TransactionStatus status)`：提交事务
  * `void rollback(TransactionStatus staus)`；回滚事务

  > 我们在开发中都是使用它的实现类，真正管理事务的对象：
  >
  > * org.springframework.jdbc.datasource.DataSourceTransactionManager：使用 Spring JDBC 或iBatis 进行持久化数据时使用
  > * org.springframework.orm.hibernate5.HibernateTransactionManager：使用Hibernate 版本进行持久化数据时使用

* `TransactionDefinition`:它是事务的定义信息对象

  * String getName()：获取事务对象名称

  * Int getIsolationLevel()：获取事务隔离级别

  * Int getPropagetionBehavior()：获取事务传播行为

  * int getTimeout()：获取事务超出时间

  * Boolean isReadOnly()：获取事务是否只读  

    >读写型事务：增加、删除、修改开启事务。只读型事务：执行查询时，也会开启事务

* `TransactionStatus`：此接口提供的是事务具体运行状态

  * void flush()：刷新事务
  * boolean hasSavepoint()：获取是否存在存储点
  * boolean isCompleted()：获取事务是否完成
  * boolean isNewTransaction()：获取事务是否为新的事务
  * boolean isRollbackOnly()：获取事务是否会滚
  * void setRollbackOnly()：设置事务回滚

#### 1.4 事务的传播行为

* **事务的传播行为就是多个事务方法调用时，如何定义方法间事务的传播**

* `Propagation`支持7种不同的传播机制:

  * **REQUIRED**：业务方法需要在一个事务中运行，如果方法运行时，已处于一个事务中，那么久加入到该事务中，否则自己创建一个新的事务。**这是Spring 默认的传播行为**。
  * **SUPPORTS**：如果业务方法在某个事务范围内被调用，则方法成为该事务的一部分，如果业务方法在事务范围外被待用，则方法在没有事务的环境下执行。
  * **MANDATORY**：只能在一个已存在事务中执行，业务方法不能发起自己的事务，如果业务方法在无事务的环境下调用，就抛异常
  * **REQUIRES_NEW**：业务方法总是会为自己发起一个新的事务，如果方法已运行在一个事务中，则原油事务被挂起，新的事务被创建，知道方法结束，新事物才结束，原先的事务才会恢复执行。
  * **NOT_SUPPORTED**：声明方法需要事务，如果方法没有关联到一个事务，容器不会为它开启事务。如果方法在一个事务中被调用，该事务会被挂起，在方法调用结束后，原先的事务便会恢复执行。
  * **NEVER**：声明方法绝对不能在事务范围内执行，如果放啊放在某个事务范围内执行，容器就抛异常。只有没关联到事务，才正常执行。
  * **NESTED**：如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动的事务，则按REQUIRED 属性执行。它使用了一个单独的事务，这个事务拥有多个可以会滚的保证点。内部事务会滚不会对外部事务造成影响，它只对 DateSourceTransactionManager事务管理器起效。

  >【扩展】REQUIRED_NEW和NESTED两种传播机制的区别？
  >
  >* REQUIRED_NEW：内部的事务独立运行，在各自的作用域中，可以独立的会滚或者提交；而外部的事务将不受内部事务的会滚状态的影响。
  >* NESTED：基于单一的事务来管理，提供了东哥保存点。这种多个保存点的机制允许内部事务的变更触发外部事务的回滚。而外部事务在回滚之后，仍能继续进行事务处理，即使部分操作已经被会滚。由于这个设置给予 JDBC 的保存点，所以只能在JDBC的机制执行。
  >
  >由此可知，两者都是事务嵌套，不同之处在于，内部事务之间是否存在彼此之间的影响；NESTED之间会受到影响，而产生部分回滚，而REQUIRED_NEW则是独立的。

#### 1.5 事务的隔离级别：

隔离级别反映事务提交并发访问时的处理态度，`TransactionDefinition`接口中定义了五个表示隔离级别的常量：

* ISOLATION_DEFAULT：默认级别，归属下列某一种

* ISOLATION_READ_UNCOMMITTED：是最低的事务隔离级别，它允许另一事务看到这个事务未提交的数据。

* ISOLATION_READ_COMMITTED：保证一个事务提交后才能被另一事务读取。另一事务不能读取该事务未提交的数据。

* ISOLATION_REPEATABLE_READ：这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻想读。它除了保证一个事务不能被另外一个事务读取未提交的数据以外还避免了一下情况的产生(不可重复读)

* ISOLATION_SERIALIZABLE：这个花费最高代价但是最可靠的事务隔离级别。事务被处理未顺序执行。除了防止脏读，不可重复读之外，还避免了幻象读。

  > `脏读`：指当一个事务正在访问数据，并且对数据进行了修改，而这种数据还没有提交到数据库中，这时，另一个事务也访问这个数据，然后使用了这个数据。因为这个数据还没有提交那么另一个事务读取到这个数据我们称为脏数据。依据脏数据所做的操作肯定是不正确的。
  >
  > `不可重复读`：只在一个事务内，多次读取同一数据。在这个事务还没执行结束，另一个事务也访问该同一数据，那么在第一个事务中的两次连续读取数据之间，由于第二个事务的修改第一个事务两次读物的数据可能不一样的，这样就发生了在同一个事务内的两次读取数据是不一样的，这种情况被称为不可重复读。
  >
  > `幻象读`：一个事务先后读取一个范围的记录，但两次读取的记录数不同，我们称之为幻想读(两次执行同一条select 语句会出现不同的结果，第二次读取回增加一数据行，并没有说这个两次执行是同一个事务中)

#### 1.6 Spring事务回滚规则

指示Spring 事务管理器回滚一个事务的推荐方法时在当前事务的上下文内抛出异常。Spring 事务管理器会捕获未处理的异常，然后依据规则决定是否回滚抛出异常的事务。

我们先回顾一下异常的架构

* 异常的继承结构：Error和RuntimeException及其子类成为未检查异常（unchecked），其它异常成为已检查异常（checked）。
* <img src="/img/images/spring/05/02-exception.png" alt="02-exception" />

默认配置下，Spring 只有在抛出异常为运行时unchecked异常时才回滚该事务，也就是抛出异常为RuntimeException 的子类(Errors也会导致事务回滚)，而抛出checked异常则不会导致事务回滚。可以明确的配置在抛出异常时回滚事务，`包括checked异常`。也可以明确定义那些异常抛出时不回滚事务。还可以编程性的通过setRollbackOnly() 方法指示一个事务必须回滚，在调用完 setRollbackOnly()后你所能执行的唯一操作就是回滚。		

#### 1.7 `Spring事务实现方式`

1. 编程式管理对给予 POJO 的应用来说是唯一选择。我们需要在代码中调用 beginTransaction()、commit()、rollback()等事务管理相关的方法，这就是编程式事务管理。
2. 基于 TransactionProxyFactotyBean 的声明式事务管理
3. 基于 @Transaction 的声明式事务管理
4. 基于 Aspectj AOP 配置事务

总结：

* `编程式事务`管理使用 **TransactionTemplate** 或者直接使用底层的 `PlatformTransactionManager`。对与编程式事务管理，Spring 推荐使用 **TransactionTemplate**。 
* `声明式事务`管理建立在 **AOP** 之上的，其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大优点是不需要通过编程的方式管理事务，这样就不需要在业务代码中参杂事务管理的代码，只需要在配置文件中作相关的事务规则声明(或通过给予 @Transaction注解的方式)，便可以将事务规则应用到业务逻辑中。
* 显然声明式事务管理要优于编程式事务管理，这正是 Spring 提倡的非侵入式的开发方式。声明式事务管理使业务代码不受污染，一个普通的 POJO 对象，只需要加上注解就可以获取完全的事务支持。和编程式事务管理相比，声明式事务唯一的不足地方是，后者的最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。但是即使有这样的需求，也存在很多变通的方法，比如，可以将需要进行事务管理的代码块独立为方法等等。

##二、`Spring事务实现具体`

#### 2.1 环境搭建

1. 拷贝必要的 jar 包到工程的 lib 目录

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context</artifactId>
       <version>5.0.2.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-aop</artifactId>
       <version>5.0.2.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-aspects</artifactId>
       <version>5.0.2.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.8.7</version>
   </dependency>
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-jdbc</artifactId>
       <version>5.0.2.RELEASE</version>
   </dependency>
   ```

2. 创建 spring 的配置文件并导入约束

   ```xml
   <!-- 需要导入 aop 和 tx 两个名称空间 -->
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xmlns:tx="http://www.springframework.org/schema/tx"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
                             http://www.springframework.org/schema/beans/spring -beans.xsd
                             http://www.springframework.org/schema/tx
                             http://www.springframework.org/schema/tx/spring-tx.xsd
                             http://www.springframework.org/schema/aop
                             http://www.springframework.org/schema/aop/spring -aop.xsd">
   
   </beans>
   ```

3. 准备数据库表和实体类

   ```java
   public class Account implements Serializable {
   
       private String id; // 主键
       ……
   }
   ```

4. 编写业务层接口和实现类

   ```java
   public interface IAccountService {
   	void transfer(); //增删改 
   }
   ```

   ```java
   @Service(value = "productService")
   public class ProductServiceImpl implements ProductService {
   
       @Autowired
    private ProductMapper productMapper;
   
       @Override
       @Transactional
       public void transfer() {
           productMapper.updateProductPrice("676C5BD1D35E429A8C2E114939C5685A");
           int i = 1 /0;
           productMapper.updateProductPrice("12B7ABF2A4C544568B0A7C69F36BF8B7");
       }
   }
   ```

5. 编写数据层

   ```java
   @Component
   public interface ProductMapper {
   
       @Update("update mall_product set product_price = product_price + '1000' where id = #{id}")
       @Options(useGeneratedKeys = false)
       int updateProductPrice(@Param("id") String id);
   }
   ```

6. 在配置文件中配置业务层和持久层

   ```xml
   <!-- 配置 spring 创建容器时要扫描的包 --> <!-- 开启注解扫描，管理service和dao -->
      <context:component-scan base-package="com.silhouette.*.service"/>
      <context:component-scan base-package="com.silhouette.*.mapper"/>
      
      <!-- 数据库连接池 -->
      <context:property-placeholder location="classpath:jdbc.properties" />
      
      <!-- 数据源定义,使用 druid 连接池 -->
      <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
          <property name="username" value="${jdbc.username}" />
          <property name="password" value="${jdbc.password}" />
          <property name="url" value="${jdbc.url}" />
          <property name="driverClassName" value="${jdbc.driver}" />
      
          <!-- 生产环境请把连接值改大一点 -->
          <property name="initialSize" value="${jdbc.initialSize}" />
          <property name="minIdle" value="${jdbc.minIdle}" />
          <property name="maxIdle" value="${jdbc.maxIdle}" />
          <property name="maxActive" value="${jdbc.maxActive}" />
          <property name="maxWait" value="${jdbc.maxWait}" />
      
          <property name="validationQuery" value="${jdbc.validationQuery}" />
          <property name="logAbandoned" value="${jdbc.logAbandoned}" />
          <property name="removeAbandoned" value="${jdbc.removeAbandoned}" />
          <property name="minEvictableIdleTimeMillis" value="${jdbc.minEvictableIdleTimeMillis}" />
          <property name="timeBetweenEvictionRunsMillis" value="${jdbc.timeBetweenEvictionRunsMillis}" />
          <property name="testOnBorrow" value="${jdbc.testOnBorrow}" />
          <property name="testOnReturn" value="${jdbc.testOnReturn}" />
          <property name="testWhileIdle" value="${jdbc.testWhileIdle}" />
      </bean>
      
      <!-- 配置mybatis -->
      <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
          <property name="dataSource" ref="dataSource" />
          <property name="configLocation" value="classpath:SqlMapConfig.xml"/>
          <!--mapper.xml所在位置-->
          <property name="mapperLocations" value="classpath:mapper/**/*Mapper.xml" />
      </bean>
      
      <!-- 扫描dao接口 -->
      <bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
          <!--mapper接口所在的包-->
          <property name="basePackage" value="com.silhouette.*.mapper"/>
      </bean>
   ```

#### 2.2 `基于 TransactionProxyFactoryBean的声明式事务管理`

```xml
<!-- 配置一个事务管理器 -->
<bean id="myTracnsactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager" >
  	<!-- 注入 DataSource -->
	  <property name="dataSource" ref="dataSource"/>
</bean>

<!-- 事务代理工厂 -->
<!-- 生成事务代理对象 -->
<bean id="serviceProxy" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
		<property name="transactionManager" ref="myTracnsactionManager"/>
  	<property name="transactionAttributes">
      	<props>
          	<!-- 主要 key 是方法
										ISOLATION_DEFAULT  事务的隔离级别
										PROPAGATION_REQUIRED  传播行为
						-->
          	<prop key="add*">ISOLATION_DEFAULT,PROPAGATION_REQUIRED</prop>
            <prop key="update*">ISOLATION_DEFAULT,PROPAGATION_REQUIRED</prop>
						<!-- -Exception 表示发生指定异常回滚，+Exception 表示发生指定异常提交 -->
						<prop key="transfer">ISOLATION_DEFAULT,PROPAGATION_REQUIRED,RuntimeException</prop>
	      </props>
	  </property>
</bean>
```

#### 2.3 `基于 @Transactional 的声明式事务管理`

1. 配置事务

   ```xml
   <!-- 配置事务管理器 -->
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
   		<property name="dataSource" ref="dataSource"></property>
   </bean>
     
   <!-- 开启 spring 对注解事务的支持 -->
   <tx:annotation-driven transaction-manager="transactionManager"/>
   ```

2. @Transaction注解属性说明

   | 属性                   | 类型                               | 描述                                   |
   | ---------------------- | ---------------------------------- | -------------------------------------- |
   | value                  | String                             | 可选的限定描述符，指定使用的事务管理器 |
   | propagation            | enum：Propagation                  | 可选的事务传播行为设置                 |
   | isolation              | enum：Isolation                    | 可选的事务隔离级别设置                 |
   | readOnly               | boolean                            | 读写或者只读事务，默认读写             |
   | timeout                | Int（in seconds granularity)       | 事务草食时间设置                       |
   | rollbackFor            | Class对象数组，必须继承自Throwable | 导致事务回滚的异常类数组               |
   | rollbackForClassName   | 类名数组，必须继承自Throwable      | 导致事务会滚的异常类名字数组           |
   | noRollbackFor          | Class对象数组，必须继承自Throwable | 不会导致事务回滚的异常类数组           |
   | noRollbackForClassName | 类名数组，必须继承自Throwable      | 不会导致事务回滚的异常类名字数组       |

3. 在业务层使用 @Transactional 注解

   ```java
   @Service("accountService")
   /**
    * 该注解的属性和 xml 中的属性含义一致。该注解可以出现在接口上，类上和方法上。【优先级：方法 > 类 > 接口】
    * 	出现接口上，表示该接口的所有实现类都有事务支持。
    *   出现在类上，表示类中所有方法有事务支持
    *   出现在方法上，表示方法有事务支持。
    *       注解只能应用到public方法上，作用于protected、private时，会被忽略，也不会抛出任何异常，这是有Spring AOP的本质决定的。
    */
   @Transactional(readOnly=true,propagation=Propagation.SUPPORTS)
   public class AccountServiceImpl implements IAccountService {
     	@Autowired
   	  private IAccountDao accountDao;
     	
     	@Override
        @Transactional
        public void transfer() {
          	productMapper.updateProductPrice("676C5BD1D35E429A8C2E114939C5685A");
            int i = 1 /0;
            productMapper.updateProductPrice("12B7ABF2A4C544568B0A7C69F36BF8B7");
        }
   }
   ```

4. 测试一下

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)  //使用junit4进行测试
   @ContextConfiguration(locations = {"classpath:applicationContext-spring.xml", "classpath:SqlMapConfig.xml"}) 
   public class DemoTest {
   
       @Autowired
       private ProductService productService;
   
       @Test
       public void testTransaction1(){
           productService.transfer();
       }
   }
   ```

【扩展1】

* 如果在程序中自己捕获了异常，那么事务就不会回滚

  ```java
  @Transactional(rollbackFor = Exception.class)
  //@Transactional(noRollbackFor= {ArithmeticException.class,RuntimeException.class})
  public void transfer() {
      /*productMapper.updateProductPrice("676C5BD1D35E429A8C2E114939C5685A");
            int i = 1 /0;
            productMapper.updateProductPrice("12B7ABF2A4C544568B0A7C69F36BF8B7");*/
      try {
          productMapper.updateProductPrice("676C5BD1D35E429A8C2E114939C5685A");
          int i = 1 /0;
          productMapper.updateProductPrice("12B7ABF2A4C544568B0A7C69F36BF8B7");//前面的操作会回滚
      } catch (Exception e){
        	e.printStackTrace();
      }
  }
  ```

* 原因分析：

  * 在spring的配置文件中，如果数据源的defaultAutoCommit设置为True了，那么方法中如果自己**捕获了异常**，**事务**是**不会回滚**的，如果**没有自己捕获异常则事务会回滚**。

    ```xml
    <!-- 开启 spring 对注解事务的支持 -->
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true" />
    ```

  * 可能你会发现你并没有配置这个参数，但是我们可以看我们使用的数据库连接池，这里以 Druid连接池为例。

    ```java
    /**
     * 查看源码可知
     */
    public abstract class DruidAbstractDataSource extends WrapperAdapter implements DruidAbstractDataSourceMBean, DataSource, DataSourceProxy, Serializable {
      	……
        protected volatile boolean defaultAutoCommit = true;  
        ……
    }
    ```

* 解决方法，手动回滚事务

  ```java
  @Transactional(rollbackFor = Exception.class)
  //@Transactional(noRollbackFor= {ArithmeticException.class,RuntimeException.class})
  public void transfer() {
      /*productMapper.updateProductPrice("676C5BD1D35E429A8C2E114939C5685A");
            int i = 1 /0;
            productMapper.updateProductPrice("12B7ABF2A4C544568B0A7C69F36BF8B7");*/
      try {
          productMapper.updateProductPrice("676C5BD1D35E429A8C2E114939C5685A");
          int i = 1 /0;
          productMapper.updateProductPrice("12B7ABF2A4C544568B0A7C69F36BF8B7");//前面的操作会回滚
      } catch (Exception e){
          e.printStackTrace();
          TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
      }
  }
  ```

> 推荐：对于要进行事务的方法，不使用 try catch，在上层调用时处理异常。如果上层调用时不想处理异常，则可以在catch块中增加`TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();`语句进行手动回滚。

【扩展2】

* Spring声明式事务默认会对非检查型异常和运行时异常进行回滚，而对检查型异常不进行回滚操作。`对默认规则进行修改`

  1. 让checked异常也进行回滚

     ```java
     @Transactional(rollbackFor = Exception.class)
     ```

  2. 让unchecked异常也不回滚

     ```java
     @Transactional(noRollbackFor= RuntimeException.class)
     ```

  3. 不需要事务

     ```java
     @Transactional(propagation=Propagation.NOT_SUPPORTED)
     ```

  4. 如果不添加rollbackFor等属性，Spring碰到Unchecked Exceptions都会回滚，不仅是RuntimeException，也包括Error。

【扩展3】

* 不使用 xml 的配置方式配置【全注解】

  ```java
  /**
   * 和连接数据库相关的配置类
   */
  public class JdbcConfig {
  
      @Value("${jdbc.driver}")
      private String driver;
  
      @Value("${jdbc.url}")
      private String url;
  
      @Value("${jdbc.username}")
      private String username;
  
      @Value("${jdbc.password}")
      private String password;
  
      /**
       * 创建JdbcTemplate
       * @param dataSource
       * @return
       */
      @Bean(name="jdbcTemplate")
      public JdbcTemplate createJdbcTemplate(DataSource dataSource){
          return new JdbcTemplate(dataSource);
      }
  
      /**
       * 创建数据源对象
       * @return
       */
      @Bean(name="dataSource")
      public DataSource createDataSource(){
          DriverManagerDataSource ds = new DriverManagerDataSource();
          ds.setDriverClassName(driver);
          ds.setUrl(url);
          ds.setUsername(username);
          ds.setPassword(password);
          return ds;
      }
  }
  ```

  ```java
  public class TransactionConfig {
      /**
       * 用于创建事务管理器对象
       * @param dataSource
       * @return
       */
      @Bean(name="transactionManager")
      public PlatformTransactionManager createTransactionManager(DataSource dataSource){
          return new DataSourceTransactionManager(dataSource);
      }
  }
  ```

  ```java
  /**
   * spring的配置类，相当于bean.xml
   */
  @Configuration
  @ComponentScan("com.silhouette")
  @Import({JdbcConfig.class,TransactionConfig.class})
  @PropertySource("jdbc.properties")
  @EnableTransactionManagement
  public class SpringConfiguration {
  }
  ```

#### 2.4 `基于 Aspectj AOP 配置事务`

1. 事务配置

   ```xml
   <!-- 配置事务管理器 -->
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
   		<property name="dataSource" ref="dataSource"></property>
   </bean>
   
   <!-- 事务的配置 -->
   <tx:advice id="txAdvice" transaction-manager="transactionManager">
       <tx:attributes>
           <!-- 指定方法名称：指的是业务核心方法
                read-only：是否是只读事务。默认 false，不只读。
                isolation：指定事务的隔离级别。默认值是使用数据库的默认隔离级别。
                propagation：指定事务的传播行为。
                timeout：指定超时时间。默认值为：-1。永不超时。
                rollback-for：用于指定一个异常，当执行产生该异常时，事务回滚。产生其他异常，事务不回滚。没有默认值，任何异常都回滚。
                no-rollback-for ：用于指定一个异常，当产生该异常时，事务不回滚， 产生其他异常时， 事务回滚。没有默认值，任何异常都回滚。
           -->
           <tx:method name="transfer" isolation="DEFAULT" propagation="REQUIRED"/>
           <!--<tx:method name="transfer" isolation="DEFAULT" propagation="REQUIRED" 
   													rollback-for="com.silhouette.framework.exception.BaseTxException"/>-->
       </tx:attributes>
   </tx:advice>
   
   <!-- 配置 aop -->
   <aop:config>
   		<!-- 配置切入点表达式 -->
   		<aop:pointcut expression="execution(* com.silhouette.service.impl.*.*(..))" id="pt1"/>
   		<aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"/>
   </aop:config>
   ```

2. 测试

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)  //使用junit4进行测试
   @ContextConfiguration(locations = {"classpath:applicationContext-spring.xml", "classpath:SqlMapConfig.xml"}) 
   public class DemoTest {
   
       @Autowired
       private ProductService productService;
   
       @Test
       public void testTransaction1(){
           /*
            * 注意：该方法已交由Spring进行事务管理
            *    <tx:method name="transfer" isolation="DEFAULT" propagation="REQUIRED"/>
            * 或者统一一样处理
            * <tx:method name="*" isolation="DEFAULT" propagation="REQUIRED" 
            * 										rollback-for="com.silhouette.framework.exception.BaseAppException"/>
            */
           productService.transfer();
       }
   }
   ```

3. 【扩展】程序中自己捕获了异常，那么事务就不会回滚

   * 示例代码：

     ```java
     @Override
     public void transfer() {
         try {
             productMapper.updateProductPrice("676C5BD1D35E429A8C2E114939C5685A");
             int i = 1 /0;
             productMapper.updateProductPrice("12B7ABF2A4C544568B0A7C69F36BF8B7");
         } catch (Exception e){
             e.printStackTrace();
         }
     }
     ```

   * 原因分析：

     * Spring的AOP即声明式事务管理默认是针对 unchecked exception回滚。也就是默认对RuntimeException()异常或是其子类进行事务回滚；checked异常,即Exception可try{}捕获的不会回滚，
     * 如果使用try-catch捕获抛出的unchecked异常后没有在catch块中采用页面硬编码的方式使用spring api对事务做显式的回滚，则事务不会回滚。

   * 解决方法 -- 在针对事务的类中抛出RuntimeException异常，而不是抛出Exception。

     ```java
     @Override
     public void transfer() {
         try {
             productMapper.updateProductPrice("676C5BD1D35E429A8C2E114939C5685A");
             int i = 1 /0;
             productMapper.updateProductPrice("12B7ABF2A4C544568B0A7C69F36BF8B7");
         } catch (Exception e){
             e.printStackTrace(); 
             throw new RuntimeException();
         }
     }
     ```

#### 2.5 事务回滚总结

* 事务部回滚原因分析

  * 因为编程式事务都是手动回滚的，所以不会出现事务不回滚的情况，即不回滚都是出现在声明式事务中的。
  * 声明式事务回滚的原理是：`当被切面切中或者是加了注解的方法中抛出了RuntimeException异常时，Spring会进行事务回滚。默认情况下是捕获到方法的RuntimeException异常，也就是说抛出只要属于运行时的异常（即RuntimeException及其子类）都能回滚；但当抛出一个不属于运行时异常时，事务是不会回滚的。`
  * 常见不回滚的场景：
    * 声明式事务配置切入点表达式写错了，没切中Service中的方法
    * Service方法中，把异常给try catch了，但catch里面只是打印了异常信息，没有手动抛出RuntimeException异常
    * Service方法中，抛出的异常不属于运行时异常（如IO异常），因为Spring默认情况下是捕获到运行时异常就回滚

* 保证事务回滚的方式

  1. 声明式事务配置切入点表达式写错了，没切中Service中的方法

  2. 如果Service层会抛出不属于运行时异常也要能回滚，那么可以将Spring默认的回滚时的异常修改为Exception，这样就可以保证碰到什么异常都可以回滚。具体的设置方式也说下：

     * 声明式事务，在配置里面添加一个rollback-for，代码如下：

       ```xml
        <tx:method name="update*" propagation="REQUIRED" rollback-for="java.lang.Exception"/> 
       ```

     * 注解事务，直接在注解上面指定，代码如下：

       ```java
       @Transactional(rollbackFor=Exception.class)
       ```

  3. 只有非只读事务才能回滚的，只读事务是不会回滚的。

  4. 如果在Service层用了try catch，在catch里面再抛出一个 RuntimeException异常，这样出了异常才会回滚

  5. 可以在将方式4 中的抛出异常改为，在catch块中添加手动回滚的语句`TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();`

     >注意：该方式的前提是必须开启注解式事务的支持

>*如有任何知识产权、版权问题或理论错误，还请指正。*





