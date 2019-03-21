# 观察者模式在spring事件-监听机制中的应用

> 在业务流程中，常常需要在某个事务完成后，触发另一个事务。为了降低系统的耦合性，可通过发布事件，监听者在监听到这个事件后，触发另一个事务。这就是spring的事件-监听机制。
>
> > > spring的事件-监听机制采用经典的观察者模式。先来看看观察者模式的相关代码。

## 观察者模式者的3个角色

* 观察者 。观察者 维护监听方法，当监听的事件发生时，该方法将会被处罚。

* ~~~java
  // 天气预报员
  public class Hobbits implements WeatherObserver {
  
    private static final Logger LOGGER = LoggerFactory.getLogger(Hobbits.class);
  // 监听方法，发生事件时，此方法将触发
    @Override
    public void update(WeatherType currentWeather) {
      switch (currentWeather) {
        case COLD:
          LOGGER.info("The hobbits are shivering in the cold weather.");
          break;
        case RAINY:
          LOGGER.info("The hobbits look for cover from the rain.");
          break;
        case SUNNY:
          LOGGER.info("The happy hobbits bade in the warm sun.");
          break;
        case WINDY:
          LOGGER.info("The hobbits hold their hats tightly in the windy weather.");
          break;
        default:
          break;
      }
    }
  }
  ~~~

* 

* 被观察者。被观察者维护一个观察者的列表。 当被观察者的某个状态发生改变时，将依次调用观察者的监听方法。

  ~~~java
  // 天气属于被观察者
  public class Weather {
  
    private static final Logger LOGGER = LoggerFactory.getLogger(Weather.class);
  
    private WeatherType currentWeather;
      
      // 维护了 一个 观察者列表
    private List<WeatherObserver> observers;
  
    public Weather() {
      observers = new ArrayList<>();
      currentWeather = WeatherType.SUNNY;
    }
  
    public void addObserver(WeatherObserver obs) {
      observers.add(obs);
    }
  
    public void removeObserver(WeatherObserver obs) {
      observers.remove(obs);
    }
  
    /**
     * Makes time pass for weather
     */
      // 触发事件
    public void timePasses() {
      WeatherType[] enumValues = WeatherType.values();
      currentWeather = enumValues[(currentWeather.ordinal() + 1) % enumValues.length];
      LOGGER.info("The weather changed to {}.", currentWeather);
      notifyObservers();
    }
  
      // 遍历观察者，其实调用了观察者的监听方法
    private void notifyObservers() {
      for (WeatherObserver obs : observers) {
        obs.update(currentWeather);
      }
    }
  }
  ~~~

  

* 事件的触发者。

  ~~~nsis
  在例子中，事件的触发者与被观察者写在一起，即timepasses() 
  ~~~

  

   

  

  ## 对应spirng的事件-监听机制，各角色是这样的：

  * 观察者：即监听者，顶级接口为ApplicationListener

    ~~~java
    public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
        // 当发布事件时，此方法将会触发
        void onApplicationEvent(E var1);
    }
    ~~~

    

  * 被观察者，即事件，顶级类为ApplicationEvent,这是一个抽象类

    ~~~java
    
    public abstract class ApplicationEvent extends EventObject {
        private static final long serialVersionUID = 7099057708183571937L;
        private final long timestamp = System.currentTimeMillis();
    
        public ApplicationEvent(Object source) {
            super(source);
        }
    
        public final long getTimestamp() {
            return this.timestamp;
        }
    }
    ~~~

    __可以看到此被观察者，并没有如天气预报的例子中，维护了一个观察者的列表。那么这个观察者列表，是到哪里去维护了呢？ spring 将这个列表维护在了下面的事件发布者中。__

  

  * 事件发布者，顶级接口为ApplicationEventPublisher,  spring 常用的工厂ApplicationContext即继承了该接口。因此，只要是spring的ApplicationContext子类，都能发布事件。

    ~~~~~~java
    //事件发布者顶级接口
    public interface ApplicationEventPublisher {
        void publishEvent(ApplicationEvent var1);
    }
    ~~~~~~

    

    下面看一段AbstractApplicationContext的代码：

    ~~~~~java
      public void publishEvent(ApplicationEvent event) {
            Assert.notNull(event, "Event must not be null");
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Publishing event in " + this.getDisplayName() + ": " + event);
            }
    
            this.getApplicationEventMulticaster().multicastEvent(event);
            if (this.parent != null) {
                this.parent.publishEvent(event);
            }
    
        }
    ~~~~~

    此处getApplicationEventMulticaster()方法返回一个ApplicationEventMulticaster,顾名思义，此为事件广播者，因为有很多监听者，所以此对象封装了监听者列表，对某一个事件进行广播式的发布。

    且看ApplicationEventMulticaster实现类:AbstractApplicationEventMulticaster, 此类中的内部类ListenerRetriver 维护了一个监听者列表

    ~~~~~~java
     private class ListenerRetriever {
            public final Set<ApplicationListener<?>> applicationListeners = new LinkedHashSet();
            public final Set<String> applicationListenerBeans = new LinkedHashSet();
            private final boolean preFiltered;
    
            public ListenerRetriever(boolean preFiltered) {
                this.preFiltered = preFiltered;
            }
    
            public Collection<ApplicationListener<?>> getApplicationListeners() {
                LinkedList<ApplicationListener<?>> allListeners = new LinkedList();
                Iterator var2 = this.applicationListeners.iterator();
    
                while(var2.hasNext()) {
                    ApplicationListener<?> listener = (ApplicationListener)var2.next();
                    allListeners.add(listener);
                }
    
                if (!this.applicationListenerBeans.isEmpty()) {
                    BeanFactory beanFactory = AbstractApplicationEventMulticaster.this.getBeanFactory();
                    Iterator var8 = this.applicationListenerBeans.iterator();
    
                    while(var8.hasNext()) {
                        String listenerBeanName = (String)var8.next();
    
                        try {
                            ApplicationListener<?> listenerx = (ApplicationListener)beanFactory.getBean(listenerBeanName, ApplicationListener.class);
                            if (this.preFiltered || !allListeners.contains(listenerx)) {
                                allListeners.add(listenerx);
                            }
                        } catch (NoSuchBeanDefinitionException var6) {
                            ;
                        }
                    }
                }
    
                OrderComparator.sort(allListeners);
                return allListeners;
            }
        }
    
    ~~~~~~

    剩下的疑问是，在发布者发布事件时，是不是调用监听者的监听方法，onApplicationEvent 方法呢？ 

    果不其然，在SimpleApplicationEvnetMulticaster中看到了发布事件的最终方法：

    ~~~~~~java
    protected void invokeListener(ApplicationListener listener, ApplicationEvent event) {
            ErrorHandler errorHandler = this.getErrorHandler();
            if (errorHandler != null) {
                try {
                    listener.onApplicationEvent(event);
                } catch (Throwable var5) {
                    errorHandler.handleError(var5);
                }
            } else {
                listener.onApplicationEvent(event);
            }
    
        }
    ~~~~~~

    最终，证实了与观察者模式的一致性。

    

    

    

    

  

  



