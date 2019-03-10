#### spring boot starter

1、引入相关包
2、自动配置

#### spring boot 规范

1、starter.jar 完成相关jar包的引入
2、autoConfigure.jar完成自动配置

#### @Value 与@ConfigurationProperties的区别

|              | @Value       | @ConfigurationProperties           |
| ------------ | ------------ | ---------------------------------- |
| 功能         | 一个一个指定 | 批量注入配置文件中的属性           |
| 松散语法     | 不支持       | 支持，如fistName和first-name都可以 |
| SpEL         | 支持         | 不支持                             |
| JSR303校验   | 不支持       | 支持 javax.validation              |
| 复杂类型封装 | 不支持       | 支持                               |

@Value 一个个指定属性，不支持复杂类型封装，支持SpEL

@ConfigurationProperties 批量注入属性，支持复杂类型封装，不支持SpEL
	可以配合Validation使用

@EnableConfigurationProperties

@PropertySource：加载指定的配置文件；

@ImportResource：导入Spring的配置文件，让配置文件里面的内容生效；
	@ImportResource(locations = {"classpath:beans.xml"})



#### **配置文件**

1、多profile文件形式：
– 格式：application-{profile}.properties/yml：
application-dev.properties、application-prod.properties
2、多profile文档块模式：
3、激活方式：
– 命令行 --spring.profiles.active=dev
– 配置文件 spring.profiles.active=dev
– jvm参数 –Dspring.profiles.active=dev



#### 配置文件加载位置

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文
件
–file:./config/  项目根路径
–file:./
–classpath:/config/
–classpath:/
优先级由高到底，高优先级的配置会覆盖低优先级的配置；

SpringBoot会从这四个位置全部加载主配置文件；互补配置；
我们还可以通过spring.config.location来改变默认的配置文件位置
项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默
认加载的这些配置文件共同起作用形成互补配置；
org.springframework.boot.context.config.ConfigFileApplicationListener



1. 命令行参数

   java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.properties

2. 来自java:comp/env的JNDI属性

3. Java系统属性（System.getProperties()）

4. 操作系统环境变量

5. RandomValuePropertySource配置的random.*属性值

6. jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件

7. jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件

8. jar包外部的application.properties或application.yml(不带spring.profile)配置文件

9. jar包内部的application.properties或application.yml(不带spring.profile)配置文件

10. @Configuration注解类上的@PropertySource

11. 通过SpringApplication.setDefaultProperties指定的默认属性



#### **自动配置原理**

EnableAutoConfiguration

@Conditional 扩展注解 作用（判断是否满足当前指定条件）
@ConditionalOnJava 系统的java版本是否符合要求
@ConditionalOnBean 容器中存在指定Bean；
@ConditionalOnMissingBean 容器中不存在指定Bean；
@ConditionalOnExpression 满足SpEL表达式指定
@ConditionalOnClass 系统中有指定的类
@ConditionalOnMissingClass 系统中没有指定的类
@ConditionalOnSingleCandidate 容器中只有一个指定的Bean，或者这个Bean是首选Bean
@ConditionalOnProperty 系统中指定的属性是否有指定的值
@ConditionalOnResource 类路径下是否存在指定资源文件
@ConditionalOnWebApplication 当前是web环境
@ConditionalOnNotWebApplication 当前不是web环境
@ConditionalOnJndi JNDI存在指定项

#### **日志**

**日志门面** 
~~JCL（Jakarta Commons Logging）~~
SLF4j（Simple Logging Facade for Java）
~~jboss-logging~~

**日志实现**
Log4j
JUL（java.util.logging）
Log4j2
Logback

SLF4j、Log4j、Logback出自同一个人之手

Log4j2 是 apache开发的

spring 选用JCL。不过spring5.0以后，通过spring-jcl-5.1.3.RELEASE做了相应的适配

spring boot选用SLF4j和Logback

一般用 logger.path，如果logger.file 和 logger.path都不指定，只会在控制台输出

**日志框架切换**

如何让系统中所有的日志都统一到slf4j；
1、将系统中其他日志框架先排除出去；
2、用中间包来替换原有的日志框架；
3、我们导入slf4j其他的实现

**spring-boot-starter-logging切换成spring-boot-starter-log4j2**
先排除spring-boot-starter-logging， 然后添加spring-boot-starter-log4j2依赖



#### Web开发

**静态资源**

静态资源目录，具体见 ResourceProperties

classpath:META-INF/resources
classpath:resources
classpath:static
classpath:public

https://www.webjars.org/



WebMvcAutoConfiguration

WebMvcConfigurerAdapter
	WebMvcAutoConfigurationAdapter
		addResourceHandlers
		viewResolver

ContentNegotiatingViewResolver: 组合所有的视图解析器的

DelegatingWebMvcConfiguration

EmbeddedServletContainerCustomizer
EmbeddedServletContainerCustomizerBeanPostProcessor

**扩展MVC**

继承WebMvcConfigurerAdapter自己实现一个适配器， 此时容器中所有的WebMvcConfigurer都会一起起作用；

```java
//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@Configuration
public class MyWebMvcConfigurerAdapter extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/myMvc").setViewName("success");
    }
}
```

**全面接管Spring MVC**

@EnableWebMvc 完全接管spring mvc， 所有的自动配置都失效

不推荐

**拦截器**

需要在 WebMvcConfigurerAdapter 里进行注册

```java
public class MyHandlerInterceptor implements HandlerInterceptor {

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		return false;
	}

	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {

	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {

	}

}
```

**嵌入式嵌入式Servlet容器配置修改**

```java
@Component
public class MyEmbeddedServletContainerCustomizer implements EmbeddedServletContainerCustomizer {

	@Override
	public void customize(ConfigurableEmbeddedServletContainer container) {
		container.setPort(8085);
	}

}
```

**注册Servlet/Filter/Listener**

DispatcherServletAutoConfiguration

**切换其他嵌入式Servlet容器**

tomcat 默认
netty 适合长连接
undertow 适合高并发

引入spring-boot-starter-web时会自动引入tomcat

原理见： EmbeddedServletContainerAutoConfiguration

​		EmbeddedServletContainerCustomizerBeanPostProcessor

嵌入式容器定制器：EmbeddedServletContainerCustomizer 

​	ServerProperties也是容器定制器

**嵌入式Servlet容器自动配置原理**

EmbeddedServletContainerAutoConfiguration

**嵌入式Servlet容器启动原理**



**错误处理机制**

原理：
可以参照ErrorMvcAutoConfiguration；错误处理的自动配置；



#### 数据访问

**JDBC自动配置原理**

DataSourceAutoConfiguration

DataSourceConfiguration

DataSourceInitializer 可运行脚本

**druid**

```java
@ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
       return  new DruidDataSource();
    }

    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();

        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");//默认就是允许所有访问
        initParams.put("deny","192.168.15.21");

        bean.setInitParameters(initParams);
        return bean;
    }


    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");

        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));

        return  bean;
    }
```



#### 启动配置原理

几个重要的事件回调机制
配置在META-INF/spring.factories
ApplicationContextInitializer
SpringApplicationRunListener
只需要放在ioc容器中
ApplicationRunner
CommandLineRunner

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    // 设置资源，一般是主启动类
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    // 判断web应用类型
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    // 根据spring.factories获取所有的ApplicationContextInitializer配置并实例化，
    // 然后设置给SpringApplication
    setInitializers((Collection) getSpringFactoriesInstances(
        ApplicationContextInitializer.class));
    // 根据spring.factories获取所有的ApplicationListener配置并实例化，然后设置给SpringApplication
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    // 推断出主启动类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    // 从META-INF/spring.factories中获取SpringApplicationRunListener并实例化
    // 把所有的SpringApplicationRunListener实例封装成SpringApplicationRunListeners对象
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 执行所有SpringApplicationRunListener的starting方法
    listeners.starting();
    try {
        // 将命令行的参数args封装成ApplicationArguments对象
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
        // 创建并配置IOC环境对象，执行所有listener的environmentPrepared方法
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
                                                                 applicationArguments);
        configureIgnoreBeanInfo(environment);
        Banner printedBanner = printBanner(environment);
        // 创建IOC容器，
        // Servlet : AnnotationConfigServletWebServerApplicationContext
        //			 ServletWebServerApplicationContext 重写onRefresh方法，创建了Servlet容器
        //                                              重写finishRefresh方法，启动Servlet容器
        // Reactive : AnnotationConfigReactiveWebServerApplicationContext
        // Default: AnnotationConfigApplicationContext
        context = createApplicationContext();
        exceptionReporters = getSpringFactoriesInstances(
            SpringBootExceptionReporter.class,
            new Class[] { ConfigurableApplicationContext.class }, context);
        // 配备IOC容器，设置IOC环境，应用initializer的initialize方法，执行所有listener的contextPrepared方法
        // 注册单例ApplicationArguments、Banner，加载主启动类资源，执行listener的contextLoaded方法
        prepareContext(context, environment, listeners, applicationArguments,
                       printedBanner);
        // 刷新IOC容器
        refreshContext(context);
        afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                .logStarted(getApplicationLog(), stopWatch);
        }
        listeners.started(context);
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```





#### 事件机制

配置在META-INF/spring.factories
ApplicationContextInitializer

```java
public class HelloApplicationContextInitializer implements
ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
      
 System.out.println("ApplicationContextInitializer...initialize..."+applicationContext);
    }
}
```

SpringApplicationRunListener

```java
public class HelloSpringApplicationRunListener implements SpringApplicationRunListener {
    //必须有的构造器
    public HelloSpringApplicationRunListener(SpringApplication application, String[] args){
    }
    @Override
    public void starting() {
        System.out.println("SpringApplicationRunListener...starting...");
    }
    @Override
    public void environmentPrepared(ConfigurableEnvironment environment) {
        Object o = environment.getSystemProperties().get("os.name");
        System.out.println("SpringApplicationRunListener...environmentPrepared.."+o);
    }
    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextPrepared...");
    }
    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextLoaded...");
    }
    @Override
    public void finished(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("SpringApplicationRunListener...finished...");
    }
}
```

配置（META-INF/spring.factories）

```properties
org.springframework.context.ApplicationContextInitializer=\
com.atguigu.springboot.listener.HelloApplicationContextInitializer
org.springframework.boot.SpringApplicationRunListener=\
com.atguigu.springboot.listener.HelloSpringApplicationRunListener
```

只需要放在ioc容器中
ApplicationRunner

```java
@Component
public class HelloApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("ApplicationRunner...run....");
    }
}
```

CommandLineRunner

```java
@Component
public class HelloCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner...run..."+ Arrays.asList(args));
    }
}
```

#### 自定义starter

```
@Configuration  //指定这个类是一个配置类
@ConditionalOnXXX  //在指定条件成立的情况下自动配置类生效
@AutoConfigureAfter  //指定自动配置类的顺序
@Bean  //给容器中添加组件
@ConfigurationPropertie结合相关xxxProperties类来绑定相关的配置
@EnableConfigurationProperties //让xxxProperties生效加入到容器中
自动配置类要能加载
将需要启动就加载的自动配置类，配置在META‐INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
```


启动器只用来做依赖导入；
专门来写一个自动配置模块；
启动器依赖自动配置；别人只需要引入启动器（starter）
mybatis-spring-boot-starter；自定义启动器名-spring-boot-starter
