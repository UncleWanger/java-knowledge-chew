# 深入SPRING源码读书笔记

1. Environment对象，表示Profiles 以及properties， profiles 在spring配置文件中， 是内嵌beans标签的的一个属性。如果某个bean 需要感知Environment ，可以实现EnvironmentAware接口。

2. xmlBeanDefinitionReader在解析xml的时候， 分为传统的标签以及自定义的标签。对于这2种标签，解析的方式是不一样的。其中传统的标签，仅有四种，分别是： import , beans,bean, alias

3. @PostConstruct 与InitializingBean 接口：  实现 InitializingBean 的 afterPropertiesSet 方法后， 初始化完成后 ，  会执行该方法。  

   PostConstructor在构造方法执行之后执行。

   Constructor, PostConstructor,InitializingBean 执行顺序：

   Constructor>PostConstruct>InitializingBean .。

   __需要强调的是：PostConstruct需要<context:annotation-config/> 标签的支持。

   ~~~~~~xml
   //  这个配置用来激活注解能被spring探测到， 比如@Required and
   	@Autowired, as well as JSR 250's @PostConstruct, @PreDestroy and @Resource (if available),
   	JAX-WS's @WebServiceRef (if available), EJB 3's @EJB (if available), and JPA's
   	@PersistenceContext and @PersistenceUnit (if available). 这些接口。
   需要注意的是， 事务相关的注解探测需要开启 <tx:annotation-driven> 。
   <xsd:element name="annotation-config">
   		<xsd:annotation>
   			<xsd:documentation><![CDATA[
   	Activates various annotations to be detected in bean classes: Spring's @Required and
   	@Autowired, as well as JSR 250's @PostConstruct, @PreDestroy and @Resource (if available),
   	JAX-WS's @WebServiceRef (if available), EJB 3's @EJB (if available), and JPA's
   	@PersistenceContext and @PersistenceUnit (if available). Alternatively, you may
   	choose to activate the individual BeanPostProcessors for those annotations.
   
   	Note: This tag does not activate processing of Spring's @Transactional or EJB 3's
   	@TransactionAttribute annotation. Consider the use of the <tx:annotation-driven>
   	tag for that purpose.
   
   	See javadoc for org.springframework.context.annotation.AnnotationConfigApplicationContext
   	for information on code-based alternatives to bootstrapping annotation-driven support.
   			]]></xsd:documentation>
               
               
   ~~~~~~

   具体解析过程可参考： AnnotationConfigBeanDefinitionParser。



4. ​	SPRING 在初始化过程中，会创建一大批Processor，为何会创建？如ConfigurationClassPostProcessor

    EventListenerPostProcessor, 。 。 。 

   org.springframework.context.annotation.interna

   lConfigurationAnnotationProcessor

   org.springframework.boot.autoconfigure.internalCachingMetadataReaderFactory

   org.springframework.boot.autoconfigure.condition.BeanTypeRegistry

   org.springframework.boot.autoconfigure.AutoConfigurationPackages

   

   



5. BeanFactoryPostProcessor 与 BeanPostProcessor区别：BeanFactoryPostProcessor 用以更改定制化的BeanDefinition ， BeanPostProcessor 用以 处理 实例化后的bean。 



name  | age

--- | ---

row1  col1 | row1 col2

***

[baidu](http://www.baidu.com)

**这是粗体**
*这是斜体*

- kldskd
- sdksld





1. kdslkdls
2. skdlskd





```java

```

[TOC]

|name|gender|age|
||||
||



-[x] kldskdl

```sequence
loop every day 

```

