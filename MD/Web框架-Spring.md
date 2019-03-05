## Spring
### 什么是Spring
Spring是个包含一系列功能的合集，如快速开发的Spring Boot，支持微服务的Spring Cloud，支持认证与鉴权的Spring Security，Web框架Spring MVC。IOC与AOP依然是核心。

### Spring MVC流程

1. 发送请求——>DispatcherServlet拦截器拿到交给HandlerMapping
2. 依次调用配置的拦截器，最后找到配置好的业务代码Handler并执行业务方法
3. 包装成ModelAndView返回给ViewResolver解析器渲染页面

### 解决循环依赖
无参数构造器、字段注入

### Bean的生命周期

1. Spring对Bean进行实例化
2. Spring将值和Bean的引用注入进Bean对应的属性中
3. 容器通过Aware接口把容器信息注入Bean
4. BeanPostProcessor。进行进一步的构造，会在InitialzationBean前后执行对应方法，当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理
5. InitializingBean。这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。
6. DisposableBean。Bean将一直驻留在应用上下文中给应用使用，直到应用上下文被销毁，如果Bean实现了接口，Spring将调用它的destory方法

### Bean的作用域

* singleton：单例模式，Spring IoC容器中只会存在一个共享的Bean实例，无论有多少个Bean引用它，始终指向同一对象。
* prototype：原型模式，每次通过Spring容器获取prototype定义的bean时，容器都将创建一个新的Bean实例，每个Bean实例都有自己的属性和状态。
* request：在一次Http请求中，容器会返回该Bean的同一实例。而对不同的Http请求则会产生新的Bean，而且该bean仅在当前Http Request内有效。
* session：在一次Http Session中，容器会返回该Bean的同一实例。而对不同的Session请求则会创建新的实例，该bean实例仅在当前Session内有效。
* global Session：在一个全局的Http Session中，容器会返回该Bean的同一个实例，仅在使用portlet context时有效。


### IOC（DI）
控制反转：原来是自己主动去new一个对象去用，现在是由容器工具配置文件创建实例让自己用，以前是自己去找妹子亲近，现在是有中介帮你找妹子，让你去挑选，说白了就是用面向接口编程和配置文件减少对象间的耦合，同时解决硬编码的问题（XML）

依赖注入：在运行过程中当你需要这个对象才给你实例化并注入其中，不需要管什么时候注入的，只需要写好成员变量和set方法

### Spring AOP
#### 介绍
面向切面的编程，是一种编程技术，是OOP（面向对象编程）的补充和完善。OOP的执行是一种从上往下的流程，并没有从左到右的关系。因此在OOP编程中，会有大量的重复代码。而AOP则是将这些与业务无关的重复代码抽取出来，然后再嵌入到业务代码当中。常见的应用有：权限管理、日志、事务管理等。

#### 实现方式
实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。Spring AOP实现用的是动态代理的方式。

#### Spring AOP使用的动态代理原理
jdk反射：通过反射机制生成代理类的字节码文件，调用具体方法前调用InvokeHandler来处理  
cglib工具：利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理

1. 如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP
2. 如果目标对象实现了接口，可以强制使用CGLIB实现AOP
3. 如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

欢迎光临[我的博客](http://www.wangtianyi.top/?utm_source=github&utm_medium=github)，发现更多技术资源~
