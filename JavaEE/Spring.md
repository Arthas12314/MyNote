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

4. Spring和Junit整合