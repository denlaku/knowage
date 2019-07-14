### spring  事务管理的特点

Spring 框架为事务管理提供一个一套统一的抽象，带来的好处有：
1、 跨不同事务 API 的统一的编程模型，无论你使用的是 jdbc、jta、jpa、hibernate。
2、 支持声明式事务
3、 简单的事务管理 API
4、 能与 spring 的数据访问抽象层完美集成

### 事务概念学习

#### Isolation  隔离级别

此事务与其他事务的工作隔离的程度。例如，该事务能否看到来自其他事务的未提交的
写操作
READ_UNCOMMITTED 读未提交
READ_COMMITTED 读提交
REPEATABLE_READ 可重复读
SERIALIZABLE 序列化（串行）

#### Read/Write  读写

该事务操作是读、还是写、有读有写

#### Timeout  超时

对事务执行的时长设置一个阀值，如果超过阀值还未完成则回滚。

#### Propagation  传播行为

当一个方法开启事务后，在方法中调用了其他的方法，其他方法可能也需要事务管理，
此时就涉及事务该如何传播了。

1. TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；
  如果当前没有事务，则创建一个新的事务。
2. TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前
  存在事务，则把当前事务挂起。
3. TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；
  如果当前没有事务，则以非事务的方式继续运行。
4. TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前
  存在事务，则把当前事务挂起。
5. TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，
  则抛出异常。
6. TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事
  务；如果当前没有事务，则抛出异常。
7. TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作
  为 当 前 事 务 的 嵌 套 事 务 来 运 行 ； 如 果 当 前 没 有 事 务 ， 则 该 取 值 等 价 于
  TransactionDefinition.PROPAGATION_REQUIRED。

#### SavePoint  保存点

事务中可以设置一些保存点（阶段标识），回滚时可以指定回滚到前面的哪个保存点。

#### Commit/Rollback  提交/ 回滚

提交、回滚事务

### Spring  事务理 管理 API

#### @Transactional

#### TransactionDefinition

DefaultTransactionDefinition

TransactionAttribute

DefaultTransactionAttribute

#### TransactionStatus

TransactionStatus 事务状态，持有事务的状态信息。事务管理代码可通过它获取事务状态、以及显式地设置回滚（代替异常的方式）。它继承了 SavePoint 接口。在它的实现中会持有事务的很多对象：如事务对象、被挂起的事务资源等等

#### PlatformTransactionManager

PlatformTransactionManager 平台级的事务管理器，它抽象定义了事务管理行为，不同
的事务管理实现实现该接口。我们编程面向该接口。

AbstractPlatformTransactionManager

DataSourceTransactionManager

### 声明式事务

#### 标签解析

1、 TxNamespaceHandler
2、 <tx:advice>标签的解析器 TxAdviceBeanDefinitionParser
a)  思考：<tx:advice> advice 注册的会是个什么 advice?
b)  浏 览 TxAdviceBeanDefinitionParser 的 代 码 ， 了 解 <tx:advice> <tx:attributes>
<tx:method>的解析过程，产出了什么。
c)  浏览 TransactionInterceptor 的代码，重点：invoke(MethodInvocation invocation)方法
invokeWithinTransaction 方法
d)  浏览 TransactionInterceptor 的继承体系，事务处理的逻辑都在
TransactionAspectSupport 中
e)  浏览 TransactionAspectSupport
它里面有如下属性：
它的重点方法是 invokeWithinTransaction
f)  浏览 TransactionAttributeSource 接口定义

#### 注解事务

@EnableTransactionManagement 注解应该也是要完成同样的事情吧？
创建代理
getBean() 创建 Bean 实例
判断是否要创建代理
怎么判断？ Advisor 的 Pointcut 是否匹配。
事务是不是一个 advisor
使用 调用它的方法
切面增强、事务

##### 开启注解标签解析

<tx:annotation-driven transaction-manager="txManager"/>
1、 看 AnnotationDrivenBeanDefinitionParser 类的标签解析
AopAutoProxyConfigurer.configureAutoProxyCreator(element, parserContext);
里面完成了：
2、 看看 TransactionAttributeSourcePointcut 的两个 Pointcut 方法
3、 看看 AnnotationTransactionAttributeSource 的代码，它的构造方法
	findTransactionAttribute -> SpringTransactionAnnotationParser
4、 剩下的事情就交给 AOP 了

##### @EnableTransactionManagement

它是如何起作用的？
1、来看@EnableTransactionManagement 这个注解的定义：
2、看 TransactionManagementConfigurationSelector
3、 看 AutoProxyRegistrar
看它实现的接口方法：
4、 看 ProxyTransactionManagementConfiguration 类
5、 @import 在哪里被解析的，忘记了！回顾开启注解支持的内容
ContextNamespaceHandler
AnnotationConfigBeanDefinitionParser
ConfigurationClassPostProcessor
ConfigurationClassParser
org.springframework.context.annotation.ConfigurationClassParser.doProcessConfigurationClass()
ConfigurationClassParser.processImports(…)
启动的时候，解析@EnableTransactionManagement，完成事务管理相关的 AOP bean 注册
剩下的事情都交给 AOP
Xml ：
启动的时候，完成事务管理相关的 AOP bean 注册

### 事务管理-JTA

Java Transaction API：java根据XA规范提出的分布式事务处理API

分布式事务需要：
1、协调各数据源提交、回滚，及应对通信异常的管理机制
2、数据源需要支持这种机制
3、应对应用故障恢复机制

#### 开启jta

通过标签: 

```xml
<tx:jta-transaction-manager>
```

通过java配置：

```java
public JtaTransactionManager getTxManager() {
    return new JtaTransactionManager();
}
```

