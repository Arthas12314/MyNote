### 1.Spring概述

IoC 反转控制 AOP面向切片编程为内核
Spring优势：方便解耦，简化开发，AOP变成的支持，声明式事务的支持，方便程序的测试，方便继承各种优秀的框架，降低java EE API的使用难度，java源码经典学习范例
Spring体系结构	<img src="https://img-blog.csdnimg.cn/20191216224813110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjgyNTI1,size_10,color_FFFFFF,t_70" alt="Spring Framework Runtime" style="zoom: 50%;" />

### 2. IoC的概念

* jdbcDemo1 debug小记

  更换版本5.1.6->5.1.46

  ```xml
  <dependency>
  	<groupId>mysql</groupId>
  	<artifactId>mysql-connector-java</artifactId>
  	<version>5.1.46</version>
  </dependency>
  ```

程序的耦合和解耦

- 开发中应注意编译期不依赖，运行时才依赖
- 解耦思路
使用反射来创建对象，避免使用new关键字
通过读取配置文件来获取要创建的对象全限定类名

工厂模式解耦

- 创建Bean对象的工厂，Bean指可重用的组件

  ```java
  IAccountService as = (IAccountService) BeanFactory.getBean("accountService");
  ```

### 3.使用Spring的IoC解决程序的耦合

* IoC创建对象的时刻

  * ApplicationContext: 单例对象适用（实际多采用此接口      

    它在构建核心容器时，创建对象采取的策略是采用立即加载的方式。也就是说，只要一读取完配置文件马上就创建配置文件中配置的对象。

* BeanFactory创建对象的时刻

  * BeanFactory: 多例对象使用

    它在构建核心容器时，创建对象采取的策略是采用延迟加载的方式。也就是说，什么时候根据id获取对象了，什么时候才真正的创建对象

* Spring对Bean的管理细节

  * 创建Bean的三种方式

    * 第一种方式：使用默认构造函数构建

      在spring配置文件中使用bean标签，配以id和class属性之后，且没有其他属性和标签时，采用的就是默认构造函数创建bean对象，此时如果类中没有默认构造函数，则对象无法创建

    * 第二种方式： 使用普通工厂中的方法创建对象（使用某个类中的方法创建对象，并存入spring容器）

      ```xml
      <bean id="instanceFactory" class="com.itheima.factory.InstanceFactory"/>
          <bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService"/>
      ```

    * 第三种方式：使用工厂中的静态方法创建对象（使用某个类中的静态方法创建对象，并存入spring容器)

      ```xml
      <bean id="accountService" class="com.itheima.factory.StaticFactory" factory-method="getAccountService"/>
      ```

  * bean标签的scope属性：   

    作用：用于指定bean的作用范围    

    取值： 常用的就是单例的和多例的        

    * singleton：单例的（默认值）

      一个应用只有一个对象的实例。它的作用范围就是整个引用。 

      生命周期： 

      1. 对象出生：当应用加载，创建容器时，对象就被创建了。 
      2. 对象活着：只要容器在，对象一直活着。
      3.  对象死亡：当应用卸载，销毁容器时，对象就被销毁了。

      单例对象的生命周期与容器相同

    * prototype：多例的

      每次访问对象时，都会重新创建对象实例。

       生命周期：

      1. 对象出生：当使用对象时，创建新的对象实例。 
      2. 对象活着：只要对象在使用中，就一直活着。 
      3. 对象死亡：当对象长时间不用时，被 java 的垃圾回收器回收了。

    * request：作用于web应用的请求范围

    * session：作用于web应用的会话范围

    * global-session：作用于集群环境的会话范围（全局会话范围），当不是集群环境时，它就是session

    * init-method：指定类中的初始化方法名称。 destroy-method：指定类中销毁方法名称。

### 4.依赖注入（Dependency Injection)

* 概念

  * 依赖注入：Dependency Injection。

    它是 spring 框架核心 ioc 的具体实现，依赖关系的维护就称之为依赖注入。 

    我们的程序在编写时，通过控制反转，把对象的创建交给了 spring，但是代码中不可能出现没有依赖的情况。 ioc 解耦只是降低他们的依赖关系，但不会消除。例如：我们的业务层仍会调用持久层的方法。 那这种业务层和持久层的依赖关系，在使用 spring 之后，就让 spring 来维护了。 简单的说，就是坐等框架把持久层对象传入业务层，而不用我们自己去获取。

  * 三类能注入的数据

    * 基本类型和String
    * 其他bean类型（在配置文件中或者注解配置过的bean）
    * 复杂类型/集合类型

  * 三种注入方式

    * 第一种：使用构造函数提供

      使用的标签:constructor-arg    

      标签出现的位置：bean标签的内部    

      标签中的属性：     

      1. type：用于指定要注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型
      2. index：用于指定要注入的数据给构造函数中指定索引位置的参数赋值。索引的位置是从0开始
      3. name：用于指定给构造函数中指定名称的参数赋值（常用
      4. value：用于提供基本类型和String类型的数据
      5. ref：用于指定其他的bean类型数据。它指的就是在spring的Ioc核心容器中出现过的bean对象
      6. 优势：在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功
      7. 弊端：改变了bean对象的实例化方式，使我们在创建对象时，如果用不到这些数据，也必须提供

    * 第二种：使用set方法提供（更常用的方式

      涉及的标签：property 

      出现的位置：bean标签的内部

      标签的属性

      1. name：用于指定注入时所调用的set方法名称
      2. value：用于提供基本类型和String类型的数据
      3. ref：用于指定其他的bean类型数据。它指的就是在spring的Ioc核心容器中出现过的bean对象
      4. 优势：创建对象时没有明确的限制，可以直接使用默认构造函数    
      5. 弊端：如果有某个成员必须有值，则获取对象是有可能set方法没有执行。

    * 第三种：使用注解提供

### 5.Spring基于注解的IOC以及IOC的案例

1. Spring中IOC的常用注解

   * 用于创建对象的

     作用与在XML配置文件中<bean>标签实现的功能相同

     @Component 用于把当前类对象存入Spring容器中

     属性：

     * value：用于指定bean的id，默认值为当前类名，且首字母小写

     以下三个注解他们的作用和属性与Component相同，spring框架提供了明确的三层使用的注解，使三层对象更加清晰

     * Controller：一般用在表现层
     * Service：一般用在业务层
     * Repository：一般用在持久层

   * 用于注入数据的

     作用与在xml配置文件中的bean标签中<property>标签实现的功能相同

     * @Autowired: 

       作用：自动按照类型注入。只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配，就可以注入成功；如果ioc容器中没有任何bean的类型和要注入的变量类型匹配，则报错；

       如果Ioc容器中有多个类型匹配时：会出现报错

       出现位置：可以是变量上，也可以是方法上

       细节：在使用注解注入时，set方法就不是必须的了

     * @Qualifier

       作用：在按照类中注入的基础之上再按照名称注入，它在给类成员注入时不能单独使用，而是与Autowired搭配使用，但是在给方法参数注入时可以

       属性value：用于指定注入bean的id

     * @Resource

       作用：直接按照bean的id注入。它可以独立使用

       属性：

       name：用于指定bean的id。

     以上三个注入都只能注入其他bean类型的数据，而基本类型和String类型无法使用上述注解实现。

     另外，集合类型的注入只能通过XML来实现。

     * Value

       作用：用于注入基本类型和String类型的数据

       属性value：用于指定数据的值。它可以使用spring中SpEL(也就是spring的el表达式）

       SpEL的写法：${表达式}

       @Value的值有两类：
       ① ${ property : default_value }
       ② #{ obj.property? :default_value }
       第一个注入的是外部配置文件对应的property，第二个则是SpEL表达式对应的内容。 default_value即前面的值为空时的默认值。注意二者的不同，#{}里面那个obj代表对象。

   * 用于改变作用范围的

     作用与在xml配置文件中的bean标签中<scope>标签实现的功能相同

     * @Scope

       作用：用于指定bean的作用范围

       属性value：指定范围的取值。常用取值：singleton prototype

   * 与生命周期相关

     作用与在xml配置文件中的bean标签中<init-method>和<destroy-method>标签实现的功能相同
     
     * @PreDestroy
     
       作用：用于指定销毁方法
     
     * @PostConstruct
     
       作用：用于指定初始化方法

2. 案例使用xml方式与注解方式实现单表的CRUD操作（持久层技术选择：dubtils

3. 改造基于注解的IOC案例，使用纯注解的方式实现即新注解使用

   * spring中的新注解

     * Configuration

       作用：指定当前类是一个配置类

       细节：当配置类作为AnnotationConfigApplicationContext对象创建的参数时，该注解可以不写。

     * ComponentScan

       作用：用于通过注解指定spring在创建容器时要扫描的包

       属性value：它和basePackages的作用是一样的，都是用于指定创建容器时要扫描的包。

       使用此注解就等同于在xml中配置了:

       ```xml
       <context:component-scan base-package="com.itheima"></context:component-scan>
       ```

     * Bean

       作用：用于把当前方法的返回值作为bean对象存入spring的ioc容器中

       属性name：用于指定bean的id。当不写时，默认值是当前方法的名称

       细节：当我们使用注解配置方法时，如果方法有参数，spring框架会

       去容器中查找有没有可用的bean对象。

       查找的方式和Autowired注解的作用是一样的

     * PropertySource

       作用：用于指定properties文件的位置

       属性value：指定文件的名称和路径。

       关键字：classpath，表示类路径下

4. Spring和Junit整合

   * Junit单元测试中，没有main方法也能执行，junit集成了一个main方法，该方法就会查找当前测试类中的@Test注解并执行
   * Junit不检测是否使用Spring框架，因此不会读取配置文件/类、创建Spring核心容器

   * Spring整合junit的配置
     1. 导入spring整合junit的jar(坐标)
     2. 使用Junit提供的一个注解把原有的main方法替换了，替换成spring提供的@Runwith
     3. 告知spring的运行器，spring和ioc创建是基于xml还是注解的，并且说明位置@ContextConfiguration
        * locations：指定xml文件的位置，加上classpath关键字，表示在类路径下
        * classes：指定注解类所在地位置

5. 动态代理

* 特点：字节码随用随创建，随用随加载

* 作用：不修改源码的基础上对方法增强

* 分类：

  * 基于子类的动态代理

    * 涉及的类：Enhancer

    * 提供者：第三方cglib库

    * 如何创建代理对象：使用Enhancer类中的create方法

    * 创建代理对象的要求：被代理类不能是最终类

    * create方法的参数：

      1. Class：字节码

         它是用于指定被代理对象的字节码。

      2. Callback：用于提供增强的代码

         它是让我们写如何代理。我们一般都是些一个该接口的实现类，通常情况下都是匿名内部类，但不是必须的。

         我们一般写的都是该接口的子接口实现类：MethodInterceptor 

         此接口的实现类都是谁用谁写

  * 基于接口的动态代理：

    * 涉及的类：

    * 如何创建代理对象：使用Proxy类中的newProxyInstance方法

    * 创建代理对象的要求：被代理类最少实现一个接口，如果没有则不能使用  

    * newProxyInstance方法的参数：

      1. ClassLoader：类加载器

         它是用于加载代理对象字节码的。和被代理对象使用相同的类加载器。固定写法。

      2. Class[]：字节码数组

         它是用于让代理对象和被代理对象有相同方法。固定写法。*      

      3. InvocationHandler：用于提供增强的代码

         它是让我们写如何代理。我们一般都是些一个该接口的实现类，通常情况下都是匿名内部类，但不是必须的。此接口的实现类都是谁用谁写。

6. AOP的概念

* Joinpoint(连接点): 所谓连接点是指那些被拦截到的点。在 spring 中,这些点指的是方法,因为 spring 只支持方法类型的连接点。 
* Pointcut(切入点): 所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义。
* Advice(通知/增强): 所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知。通知的类型：前置通知,后置通知,异常通知,最终通知,环绕通知。 

* Introduction(引介): 引介是一种特殊的通知在不修改类代码的前提下, Introduction 可以在运行期为类动态地添加一些方法或 Field。 
* Target(目标对象): 代理的目标对象。 
* Weaving(织入): 是指把增强应用到目标对象来创建新的代理对象的过程。 spring 采用动态代理织入，而 AspectJ 采用编译期织入和类装载期织入。 
* Proxy（代理）: 一个类被 AOP 织入增强后，就产生一个结果代理类。 
* Aspect(切面): 是切入点和通知（引介）的结合。

6. Spring中基于xml和注解的AOP配置

* spring中基于XML的AOP配置步骤

  1. 把通知Bean也交给spring来管理

  2. 使用aop:config标签表明开始AOP的配置

  3. 使用aop:aspect标签表明配置切面           

     id属性：是给切面提供一个唯一标识

     ref属性：是指定通知类bean的Id。    

  4. 在aop:aspect标签的内部使用对应标签来配置通知的类型

     我们现在示例是让printLog方法在切入点方法执行之前之前：所以是前置通知 

     aop:before：表示配置前置通知

     * method属性：用于指定Logger类中哪个方法是前置通知                
     * pointcut属性：用于指定切入点表达式，该表达式的含义指的是对业务层中哪些方法增强        

     切入点表达式的写法：

     * 关键字：execution(表达式) 

     * 表达式：

       访问修饰符  返回值  包名.包名.包名...类名.方法名(参数列表)            标准的表达式写法：                

       ​	public void com.itheima.service.impl.AccountServiceImpl.saveAccount()            访问修饰符可以省略

       ​	void com.itheima.service.impl.AccountServiceImpl.saveAccount()            返回值可以使用通配符，表示任意返回值

       ​	\* com.itheima.service.impl.AccountServiceImpl.saveAccount()            包名可以使用通配符，表示任意包。但是有几级包，就需要写几个

       ​	\* \*.\*.\*.\*.AccountServiceImpl.saveAccount())

       包名可以使用..表示当前包及其子包 

       ​	\* \*..AccountServiceImpl.saveAccount() 

       类名和方法名都可以使用*来实现通配

       ​	\*..\*.\*() 

       参数列表：可以直接写数据类型：

       1. 基本类型直接写名称           int
       2. 引用类型写包名.类名的方式   java.lang.String 

       可以使用通配符表示任意类型，但是必须有参数

       可以使用..表示有无参数均可，有参数可以是任意类型

       全通配写法：

       ​	\*..\*.\*(..)            

       实际开发中切入点表达式的通常写法：

       切到业务层实现类下的所有方法

       ​	\* com.itheima.service.impl.\*.\*(..)

* @Pointcut 

  作用： 指定切入点表达式 

  属性：

  ​	value：指定表达式的内容 

  ```java
  @Pointcut("execution(* com.itheima.service.impl.*.*(..))") 
  private void pt1() {}
  
  引用方式：
  /**
  * 环绕通知
  * @param pjp
  * @return
  */
  @Around("pt1()")//注意：千万别忘了写括号
  public Object transactionAround(ProceedingJoinPoint pjp) {
      //定义返回值
      Object rtValue = null;
      try {
          //获取方法执行所需的参数
          Object[] args = pjp.getArgs();
          //前置通知：开启事务
          beginTransaction();
          //执行方法
          rtValue = pjp.proceed(args);
          //后置通知：提交事务
          commit();
  	}catch(Throwable e) {
          //异常通知：回滚事务
          rollback();
          e.printStackTrace();
  	}finally {
          //最终通知：释放资源
          release();
  	}
  	return rtValue;
  }
  ```

  