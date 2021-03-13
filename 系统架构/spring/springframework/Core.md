
# IoC(Inversion of Control)
> IoC is alse known as dependency injection(DI)

> 基础包
+ org.springframework.beans
+ org.springframework.context

> bean工厂
+ BeanFactory
    + ApplicationContext
        + ClassPathXmlApplicationContext
        + FileSystemXmlApplicationContext

## Annotation-based configuration:
+ @Required
+ @Autowired
+ 

## Java-based configuration
+ @Configuration
+ @Bean
+ @Import
+ @DependsOn

## Instantiation a Container
~~~java
    ApplicationContext context = new ClassPathXmlApplicationContext("***.xml",***);
~~~

## Bean
+ Class
+ Name
+ Scope
    + singleton
    + prototype
    + request
    + session
    + application
    + websocket
    
+ ConstructorArguments
+ Properties
+ AutowiringMode
+ LazyInitializationMode
+ InitalizationMethod
+ DestructionMethod




