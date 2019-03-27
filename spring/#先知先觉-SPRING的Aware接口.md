# 先知先觉-SPRING的Aware接口

## .	前言

​	书到今生读已迟。讲的是，有些人的聪明才智从天生的， 很多知识就像在上辈子就已经学过了。 Spring体系中就有类似的以***Aware结尾的接口或者类。这些类自诞生之日起，就拥有了感知某些资源的能力，并且能在本类中，使用这些资源。  aware为感知， 如ApplicationContextAware, 即有感知applicationContext的能力。 BeanFactoryAware有感知BeanFactory的能力。这些接口大体是这样使用：

1. 维护一个被感知对象的实例如： applicationContext。

2. 维护一个setApplicationContext(ApplicationContext applicationContext) 方法，将applicationContext赋值给本类中的applicationContext。 

   那么spring是如何做到的呢？ 

## .    疑问

1. spring在什么时候将感知的对象赋值到这些Aware类中呢？ 
2. 有哪些常用的Aware类？



## . 源码释疑

. Aware接口定义

~~~~~~java
/**
 * A marker superinterface indicating that a bean is eligible to be notified by the
 * Spring container of a particular framework object through a callback-style method.
 * The actual method signature is determined by individual subinterfaces but should
 * typically consist of just one void-returning method that accepts a single argument.
 *
 * <p>Note that merely implementing {@link Aware} provides no default functionality.
 * Rather, processing must be done explicitly, for example in a
 * {@link org.springframework.beans.factory.config.BeanPostProcessor}.
 * Refer to {@link org.springframework.context.support.ApplicationContextAwareProcessor}
 * for an example of processing specific {@code *Aware} interface callbacks.
 *
 这是一个标志性的接口，用以标明愿意接受spring容器或者其他特定容器通过回调方式的方法发出的通知。
 真正的签名方法在子接口中去定义， 这个签名方法接受一个参数，并且不返回。
 
 ##  以下翻译应该有误， 可请教他人。。。。
 注意几乎所有的Aware实现类都提供默认的功能。也就是说，这个处理必须是 显式 的进行。就像实现了`BeanPostProcessor`的实现类，必须要实现process方法。
 
 * @author Chris Beams
 * @author Juergen Hoeller
 * @since 3.1
 */
public interface Aware {

}
~~~~~~



. 实现aware接口的类，是在什么时候被spring容器赋予 感知的能力的？

主要有如下几处：

1. org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods

   ~~~~~~java
   private void invokeAwareMethods(final String beanName, final Object bean) {
   		if (bean instanceof Aware) {
   			if (bean instanceof BeanNameAware) {
   				((BeanNameAware) bean).setBeanName(beanName);
   			}
   			if (bean instanceof BeanClassLoaderAware) {
   				ClassLoader bcl = getBeanClassLoader();
   				if (bcl != null) {
   					((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
   				}
   			}
   			if (bean instanceof BeanFactoryAware) {
   				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
   			}
   		}
   	}
   ~~~~~~

   调用上面的方法是在spring初始化bean的时候调用的：

   ~~~~~~~java
   protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
   		if (System.getSecurityManager() != null) {
   			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
   				invokeAwareMethods(beanName, bean);
   				return null;
   			}, getAccessControlContext());
   		}
   		else {
   			invokeAwareMethods(beanName, bean);
   		}
   
   		Object wrappedBean = bean;
   		if (mbd == null || !mbd.isSynthetic()) {
   			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
   		}
   
   		try {
   			invokeInitMethods(beanName, wrappedBean, mbd);
   		}
   		catch (Throwable ex) {
   			throw new BeanCreationException(
   					(mbd != null ? mbd.getResourceDescription() : null),
   					beanName, "Invocation of init method failed", ex);
   		}
   		if (mbd == null || !mbd.isSynthetic()) {
   			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
   		}
   
   		return wrappedBean;
   	}
   
   ~~~~~~~

   在spring初始化bean时，如果发现该bean实现了BeanNameAware,或者BeanClassLoaderAware,或者BeanFactoryAware，那么会将相应的对象注入该实现类中。

   

   这里有个疑问，什么样的bean 能够被spring所管理？ 

   是在xml里配置了bean标签的对象吗？ 那如果是spring boot中，哪些类又能被spring管理呢？ 是在指定包名下所有的类吗

   2. ​	org.springframework.context.support.ApplicationContextAwareProcessor#invokeAwareInterfaces

      ~~~~~~java
      private void invokeAwareInterfaces(Object bean) {
      		if (bean instanceof Aware) {
      			if (bean instanceof EnvironmentAware) {
      				((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
      			}
      			if (bean instanceof EmbeddedValueResolverAware) {
      				((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
      			}
      			if (bean instanceof ResourceLoaderAware) {
      				((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
      			}
      			if (bean instanceof ApplicationEventPublisherAware) {
      				((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
      			}
      			if (bean instanceof MessageSourceAware) {
      				((MessageSourceAware) bean).setMessageSource(this.applicationContext);
      			}
      			if (bean instanceof ApplicationContextAware) {
      				((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
      			}
      		}
      	}
      ~~~~~~

      上面的方法在 org.springframework.context.support.ApplicationContextAwareProcessor#postProcessBeforeInitialization中调用

      ~~~~~~java
      public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
      		AccessControlContext acc = null;
      
      		if (System.getSecurityManager() != null &&
      				(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
      						bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
      						bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)) {
      			acc = this.applicationContext.getBeanFactory().getAccessControlContext();
      		}
      
      		if (acc != null) {
      			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
      				invokeAwareInterfaces(bean);
      				return null;
      			}, acc);
      		}
      		else {
      			invokeAwareInterfaces(bean);
      		}
      
      		return bean;
      	}
      ~~~~~~

      

   这是在处理BeanPostProcessor时进行处理

   

   