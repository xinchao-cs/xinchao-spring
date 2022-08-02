# xinchao-spring
关于spring框架的知识笔记
一、spring框架的两大核心接口：BeanFactory与ApplicationContext
   ApplicationContext是BeanFactory的子接口，都是spring的容器
 1、有何区别：
   BeanFactory：是spring最底层的接口，包含了各种Bean的定义，读取Bean的配置文档，管理Bean的加载，实例化，控制Bean的生命周期，维护Bean之间的依赖关系。
   ApplicationContext：是BeanFactory接口的派生类，除了提供BeanFactory的实现功能外，还提供了更完整的框架功能：
     1.集成了MessageSource，因此支持国际化
     2.统一的资源文件访问方式
     3.提供了在监听器中注册Bean的方式
     4.可同时加载多个配置文件
     5.载入多个（有继承关系）的上下文，使得每一个上下文都专注于特定的层次，例如Web层
 2、加载方式：
   BeanFactory是采用延迟加载的方式注入Bean的，在使用Bean的时候（getBean（））才会对其进行加载实例化，就无法发现其在spring配置上的问题，如果Bean中的某个属性没有注入，知道BeanFactory记载后，第一次调用getBean（）才会抛出异常
   ApplicationContext是在容器启动后，一次性创建好所有的Bean。这样在容器启动后我们就可以发现存在于spring配置上的问题，有利于检查Bean的依赖属性是否注入。ApplicationContext启动后预载入单实例Bean，在使用的时候无需等待饥渴调用实例Bean。
   不足点：都比较占空间，当应用程序创建的Bean创建的过多的时候，会影响到程序的启动
3、创建方式：
   BeanFactory通常以编程的方式创建
   Application还能以注解的方式创建，但需要使用ContextLoader
   
二、bean的生命周期：
  四个阶段：实例化、属性赋值、初始化、销毁
  多个扩展点：影响多个bean：
               BeanPostProcessor（作用于初始化阶段的前后）
               InstantiationAwareBeanPostProcessor(作用于实例化阶段的前后)
            影响单个bean：
               Aware接口（作用是拿到spring容器中的一些资源）
                 Aware Group1
                   BeanNameAware
                   BeanClassLoaderAware
                   BeanFactoryAware
                 Aware Group2
                   EnvironmentAware
                   EmbeddedValueResolverAware(实现该接口能够获取Spring EL解析器，用户的自定义注解需要支持spel表达式的时候可以使用)
                   ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware)
           生命周期（实例化和属性赋值是spring容器去实现的，自己需要操作实现初始化和销毁两个阶段的内容）：
              InitializingBean
              DisposableBean
           bean的作用域：
              singleton
               <bean id="xxxxx" class="xxx.xxx.xxx" scope="singleton"></bean>
               存储单例，所有对Bean的请求只要与id和bean的定义相匹配，则会返回bean的同一实例，主要是为了spring的缺省作用域
              prototype
               <bean id="xxxxx" class="xxx.xxx.xxx" scope="prototype"></bean>
               每次对bean发起请求的时候，springIOC都会新建一个作用域
               对于有状态的bean作用域应该选择prototype，对于无状态的作用域应该选择singleton
              request
               <bean id="xxxxx" class="xxx.xxx.xxx" scope="request"></bean>
               Request作用域只针对于http请求的，spring容器会根据请求bean的定义创建一个全新的bean实例，而且该bean只在当前的request请求作用域中有效
              global session
               <bean id="xxxxx" class="xxx.xxx.xxx" scope="globalSession"></bean>
               类似标准的http session作用域，不过仅仅在基于portlet的web应用当中才有意义。Portlet规范定义了全局的Session的概念。他被所有构成某个portlet外部应用中的各种不同的                  portlet所共享。在global session作用域中所定义的bean被限定于全局的portlet session的生命周期范围之内。
              
三、springAOP
   AOP原理：
     1、在IOC的过程中，在创建bean的过程中，都会对bean进行处理实现增强，对于AOP来说就是创建代理类
     2、底层就是动态代理技术：
        JDK动态代理（基于接口）
        CGLib动态带你（基于类）
        在SpringAOP中，如果使用的是单例，则推荐使用CGlib动态代理类
     3、JDK动态代理;
        1.JDK的动态代理是基于反射的实现，JDK通过反射，生成一个代理类，该代理类实现了被代理类的所有的接口，并对接口中定义的所有方法都进行了代理。当通过代理类执行被代理类中的方法时，           代理类底层会通过反射机制，回调我们实现的InvocationHandler接口的invoke（）方法。该代理类是Proxy的子类。------JDK动态代理的实现方式
        2.优点：
          JDK动态代理是JDK原生的，不需要任何的依赖
          通过反射机制生成代理类的方式比CGlib操作字节码生成代理类的方式要快
        3.缺点；
          使用JDK动态代理是需要被代理类实现了接口中所声明的方法。否则无法进行代理。
          JDK动态代理无法为没有在接口中声明的方法实现代理。
          JDK动态代理在执行代理方法时，需要通过反射机制进行回调，会降低代码的执行效率。
     4、CGLib动态代理
        1.CGLib动态代理实现原理：底层是通过ASM字节码生成框架，操作被代理类的字节码，生成这个类的子类，并重写了类中所有可重写的方法，在重写的过程中，将额外的逻辑增加到重写方法中，在           织入到方法，对方法进行增强。通过代理类生成的字节码文件与自己编写后编译生成的字节码文件没太大区别
        2.优点：
          使用CGLib代理的类，不需要去实现接口，因为他是直接继承自被代理类。
          CGLib代理的类是被代理类的子类，所以能够对被代理类中所有被重写的方法进行代理
          CGLib生成的代理类与自己编写后编译生成的类没有太大区别，方法的调用与普通类一致，所以效率要更高
        3.缺点
          由于CGLib的代理类使用的是继承，这也就意味着如果需要被代理的类是一个final类，则无法使用CGLib代理；
          由于CGLib实现代理方法的方式是重写父类的方法，所以无法对final方法，或者private方法进行代理，因为子类无法重写这些方法；
          CGLib生成代理类的方式是通过操作字节码，这种方式生成代理类的速度要比JDK通过反射生成代理类的速度更慢；
        
              
