---
layout: post
title: "SpringBoot学习笔记一"
date: 2017-08-17 00:09:28 +0800
comments: true
categories: Dev
tags: [Java, SpringBoot]
---

## 特点

+ Create stand-alone Spring applications
+ Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)
+ Provide opinionated 'starter' POMs to simplify your Maven configuration
+ Automatically configure Spring whenever possible
+ Provide production-ready features such as metrics, health checks and externalized configuration
+ Absolutely no code generation and no requirement for XML configuration

<!-- more -->

SpringBoot应用Maven依赖加入
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-parent</artifactId>
    <version>1.5.6.RELEASE</version>
    <scope>import</scope>
    <type>pom</type>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>1.5.6.RELEASE</version>
</dependency>
```

如果要web环境，只需要加入web的starter
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>1.5.6.RELEASE</version>
</dependency>
```

SpringBoot应用建立
```
@Slf4j
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        app.run(args);
        //或者直接调用run方法
       //SpringApplication.run(Application.class, args);
    }     
}
```
注解@SpringBootApplication是@Configuration、@EnableAutoConfiguration 和@ComponentScan的简化方式。

### 初始化
创建SpringApplication对象实例，构造器中会调用初始化操作
```java
@SuppressWarnings({ "unchecked", "rawtypes" })
private void initialize(Object[] sources) {
   if (sources != null && sources.length > 0) {
      this.sources.addAll(Arrays.asList(sources));
   }
    // 判断是否是web程序(javax.servlet.Servlet和org.springframework.web.context.ConfigurableWebApplicationContext都必须在类加载器中存在)，
    // 并设置到webEnvironment属性中
   this.webEnvironment = deduceWebEnvironment();
   // 从spring.factories文件中找出key为ApplicationContextInitializer的类并实例化后设置到SpringApplication的initializers属性中。
   //这个过程也就是找出所有的应用程序初始化器
   setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    // 从spring.factories文件中找出key为ApplicationListener的类并实例化后设置到SpringApplication的listeners属性中。
    //这个过程就是找出所有的应用程序事件监听器
   setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
   // 找出main类，这里是Application类
   this.mainApplicationClass = deduceMainApplicationClass();
}
```

判断是否是WEB环境
```Java
private boolean deduceWebEnvironment() {
   for (String className : WEB_ENVIRONMENT_CLASSES) {
      if (!ClassUtils.isPresent(className, null)) {
         return false;
      }
   }
   return true;
}
```



ApplicationContextInitializer，应用程序初始化器，做一些初始化的工作：
```Java
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {
	void initialize(C applicationContext);
}
```

ApplicationListener，应用程序事件(ApplicationEvent)监听器：
```java
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
	void onApplicationEvent(E event);
}
```

应用程序事件(ApplicationEvent)监听器有一下几种:(org.springframework.boot.context.event)
```
ApplicationStartingEvent(应用程序启动事件)
ApplicationEnvironmentPreparedEvent(环境准备事件)
ApplicationPreparedEvent(准备事件)
ApplicationReadyEvent(启动事件)
ApplicationFailedEvent(失败事件)
```

默认情况下，initialize方法从spring.factories文件中找出的key为ApplicationContextInitializer的类有：
```
org.springframework.boot.context.config.DelegatingApplicationContextInitializer
org.springframework.boot.context.ContextIdApplicationContextInitializer
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer
org.springframework.boot.context.web.ServerPortInfoApplicationContextInitializer
org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer
```

key为ApplicationListener的有：
```
org.springframework.boot.context.config.AnsiOutputApplicationListener
org.springframework.boot.context.config.ConfigFileApplicationListener
org.springframework.boot.logging.LoggingApplicationListener
org.springframework.boot.logging.ClasspathLoggingApplicationListener
org.springframework.boot.autoconfigure.BackgroundPreinitializer
org.springframework.boot.context.config.DelegatingApplicationListener
org.springframework.boot.builder.ParentContextCloserApplicationListener
org.springframework.boot.context.FileEncodingApplicationListener
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
```

分析run方法之前，先看一下SpringApplication中的一些事件和监听器概念
> 首先是SpringApplicationRunListeners类和SpringApplicationRunListener类的介绍。

SpringApplicationRunListeners内部持有SpringApplicationRunListener集合和1个Log日志类。用于SpringApplicationRunListener监听器的批量执行。

SpringApplicationRunListener看名字也知道用于监听SpringApplication的run方法的执行。

```

started(run方法执行的时候立马执行；对应事件的类型是ApplicationStartingEvent)

environmentPrepared(ApplicationContext创建之前并且环境信息准备好的时候调用；对应事件的类型是ApplicationEnvironmentPreparedEvent)

contextPrepared(ApplicationContext创建好并且在source加载之前调用一次；没有具体的对应事件)

contextLoaded(ApplicationContext创建并加载之后并在refresh之前调用；对应事件的类型是ApplicationPreparedEvent)

finished(run方法结束之前调用；对应事件的类型是ApplicationReadyEvent或ApplicationFailedEvent)

```



### 启动
```
/**
 * Run the Spring application, creating and refreshing a new
 * {@link ApplicationContext}.
 * @param args the application arguments (usually passed from a Java main method)
 * @return a running {@link ApplicationContext}
 */
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch(); // 构造一个任务执行观察器
   stopWatch.start();   // 开始执行，记录开始时间
   ConfigurableApplicationContext context = null;
   FailureAnalyzers analyzers = null;
   configureHeadlessProperty();
   //加载META-INF/spring.factories, 获取SpringApplicationRunListeners，内部只有一个EventPublishingRunListener
   SpringApplicationRunListeners listeners = getRunListeners(args);  
   //这里接受ApplicationStartingEvent事件的listener会执行相应的操作
   listeners.starting();
   try {
      // 构造一个应用程序参数持有类
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
      //创建应用程序的环境信息。如果是web程序，创建StandardServletEnvironment；否则，创建StandardEnvironment
      //执行ApplicationEnvironmentPreparedEvent事件
      ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
      Banner printedBanner = printBanner(environment);
       // 创建Spring容器
      context = createApplicationContext();
      analyzers = new FailureAnalyzers(context);
      prepareContext(context, environment, listeners, applicationArguments, printedBanner);
      // Spring容器的刷新
      refreshContext(context);
      //// 调用Spring容器中的ApplicationRunner和CommandLineRunner接口的实现类
      afterRefresh(context, applicationArguments);
      // 广播出ApplicationReadyEvent事件给相应的监听器执行
      listeners.finished(context, null);
      stopWatch.stop();
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
      }
       // 返回Spring容器
      return context;
   }
   catch (Throwable ex) {
      handleRunFailure(context, listeners, analyzers, ex);
      throw new IllegalStateException(ex);
   }
}
```

创建容器中
Spring 5 - Spring webflux 是一个新的非堵塞函数式 Reactive Web 框架，可以用来建立异步的，非阻塞，事件驱动的服务，并且扩展性非常好

### 应用Application启动过程

创建容器阶段
```java
protected ConfigurableApplicationContext createApplicationContext() {
   Class<?> contextClass = this.applicationContextClass;
    //判断是WEB容器还是非WEB容器
   if (contextClass == null) {
      try {
         contextClass = Class.forName(this.webEnvironment
               ? DEFAULT_WEB_CONTEXT_CLASS : DEFAULT_CONTEXT_CLASS);
      }
      catch (ClassNotFoundException ex) {
         throw new IllegalStateException(
               "Unable create a default ApplicationContext, "
                     + "please specify an ApplicationContextClass",
               ex);
      }
   }
   return (ConfigurableApplicationContext) BeanUtils.instantiate(contextClass);
}
```

容器准备阶段
```
private void prepareContext(ConfigurableApplicationContext context,
      ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
      ApplicationArguments applicationArguments, Banner printedBanner) {
   // 设置Spring容器的环境信息
   context.setEnvironment(environment);
    // 回调方法，Spring容器创建之后做一些额外的事
   postProcessApplicationContext(context);
   // SpringApplication的的初始化器开始工作,批量执行ApplicationContextInitializer.initialize()方法
   applyInitializers(context);

   listeners.contextPrepared(context);  //没对应具体的事件，EventPublishingRunListener.contextPrepared()不执行任何操作
   if (this.logStartupInfo) {
      logStartupInfo(context.getParent() == null);
      logStartupProfileInfo(context);
   }

   // Add boot specific singleton beans
   context.getBeanFactory().registerSingleton("springApplicationArguments", applicationArguments);
   if (printedBanner != null) {
      context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
   }

   // Load the sources
   Set<Object> sources = getSources();
   Assert.notEmpty(sources, "Sources must not be empty");
   //加载bean到容器context中
   load(context, sources.toArray(new Object[sources.size()]));
   //执行ApplicationPreparedEvent
   listeners.contextLoaded(context);
}

## 其中
/**
 * Apply any relevant post processing the {@link ApplicationContext}. Subclasses can
 * apply additional processing as required.
 * @param context the application context
 */
protected void postProcessApplicationContext(ConfigurableApplicationContext context) {
   if (this.beanNameGenerator != null) {
      context.getBeanFactory().registerSingleton(AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR, this.beanNameGenerator);
   }
   // 如果SpringApplication设置了资源加载器，设置到Spring容器中
   if (this.resourceLoader != null) {
      if (context instanceof GenericApplicationContext) {
         ((GenericApplicationContext) context).setResourceLoader(this.resourceLoader);
      }
      if (context instanceof DefaultResourceLoader) {
         ((DefaultResourceLoader) context).setClassLoader(this.resourceLoader.getClassLoader());
      }
   }
}
```

容器刷新阶段
```
private void refreshContext(ConfigurableApplicationContext context) {
   refresh(context);
   if (this.registerShutdownHook) {
      try {
         context.registerShutdownHook();
      }
      catch (AccessControlException ex) {
         // Not allowed in some environments.
      }
   }
}

## 执行applicationContext.refresh()方法
protected void refresh(ApplicationContext applicationContext) {
   Assert.isInstanceOf(AbstractApplicationContext.class, applicationContext);
   ((AbstractApplicationContext) applicationContext).refresh();
}

## spring-context包内的applicationContext.refresh()方法
public void refresh() throws BeansException, IllegalStateException {
    Object var1 = this.startupShutdownMonitor;
    synchronized(this.startupShutdownMonitor) {
        this.prepareRefresh();
        ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
        this.prepareBeanFactory(beanFactory);

        try {
            this.postProcessBeanFactory(beanFactory);
            this.invokeBeanFactoryPostProcessors(beanFactory);
            this.registerBeanPostProcessors(beanFactory);
            this.initMessageSource();
            this.initApplicationEventMulticaster();
            this.onRefresh();
            this.registerListeners();
            this.finishBeanFactoryInitialization(beanFactory);
            this.finishRefresh();
        } catch (BeansException var9) {
            if(this.logger.isWarnEnabled()) {
                this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
            }

            this.destroyBeans();
            this.cancelRefresh(var9);
            throw var9;
        } finally {
            this.resetCommonCaches();
        }

    }
}
```

容器创建完成之后
```
/**
 * Called after the context has been refreshed.
 * @param context the application context
 * @param args the application arguments
 */
protected void afterRefresh(ConfigurableApplicationContext context, ApplicationArguments args) {
   callRunners(context, args);
}

private void callRunners(ApplicationContext context, ApplicationArguments args) {
   List<Object> runners = new ArrayList<Object>();
   runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
   runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
   // 对runners进行排序
   AnnotationAwareOrderComparator.sort(runners);

   for (Object runner : new LinkedHashSet<Object>(runners)) {
      // 找出Spring容器中ApplicationRunner接口的实现类
      if (runner instanceof ApplicationRunner) {
         callRunner((ApplicationRunner) runner, args);
      }
      // 找出Spring容器中CommandLineRunner接口的实现类
      if (runner instanceof CommandLineRunner) {
         callRunner((CommandLineRunner) runner, args);
      }
   }
}
```
这样run方法执行完成之后。Spring容器也已经初始化完成，各种监听器和初始化器也做了相应的工作。


##配置自动导入
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

主要看EnableAutoConfigurationImportSelector这个类，这个类会导入spring boot中的一个配置文件，方法如下：
```
 protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
            AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
                getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
        Assert.notEmpty(configurations,
                "No auto configuration classes found in META-INF/spring.factories. If you "
                        + "are using a custom packaging, make sure that file is correct.");
        return configurations;
 }

```
这个方法会导入spring-boot-autoconfigure包下META-INF/spring.factories配置文件spring.factories


>Spring Boot 内部提供了很多自动化配置的类，例如，
RedisAutoConfiguration 、MongoRepositoriesAutoConfiguration 、ElasticsearchAutoConfiguration ，
这些自动化配置的类会判断 classpath 中是否存在自己需要的那个类，如果存在则会自动配置相关的配置，否则就不会自动配置，
因此，开发者在 Maven 的 pom 文件中添加相关依赖后，这些依赖就会下载很多 jar 包到 classpath 中，有了这些 lib 就会触发自动化配置，
所以，我们就能很便捷地使用对于的模块功能了。

我们看一下自动加载的配置
```
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.hornetq.HornetQAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceDelegatingViewResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.SitePreferenceAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.ReactorAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.OAuth2AutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.social.SocialWebAutoConfiguration,\
org.springframework.boot.autoconfigure.social.FacebookAutoConfiguration,\
org.springframework.boot.autoconfigure.social.LinkedInAutoConfiguration,\
org.springframework.boot.autoconfigure.social.TwitterAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.velocity.VelocityAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.web.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration

# Template availability providers
org.springframework.boot.autoconfigure.template.TemplateAvailabilityProvider=\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.mustache.MustacheTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.velocity.VelocityTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.web.JspTemplateAvailabilityProvider
```
我们来看如何自动加载MongoAutoConfiguration
```
@Configuration
@ConditionalOnClass(MongoClient.class)
@EnableConfigurationProperties(MongoProperties.class)
@ConditionalOnMissingBean(type = "org.springframework.data.mongodb.MongoDbFactory")
public class MongoAutoConfiguration {

   private final MongoProperties properties;

   private final MongoClientOptions options;

   private final Environment environment;

   private MongoClient mongo;
...
```
@ConditionalOnClass(MongoClient.class)
当我们的Jar依赖也就是有我们的环境中存在MongoClient.class,才会创建这个Bean；
其中ConditionalOnClass中依赖@Conditional(OnClassCondition.class)条件，判断是否加载。



### 参考  
[Spring Boot起步依赖源码分析](http://www.imooc.com/article/15432)  
[SpringBoot源码分析之SpringBoot的启动过程](https://fangjian0423.github.io/2017/04/30/springboot-startup-analysis/)
