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
