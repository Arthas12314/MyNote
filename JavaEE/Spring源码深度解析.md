## Spring源码深度解析

### 1.Spring整体架构和环境搭建

* Spring整体架构

  * Core Container

    * Core

      Core 模决主要包含 Spring 框架基本的核心工具类，Spring 的其他纽件都要用到这个包 里的类，Core 模块是其他纽件的基本核心

    * Beans

      Beans 模块是所有应用都妥用到的，它包含访问配直文件、创建和管理 bean 以及进行 Inversion of Control/Dependency Injection ( IoC/DI ）操作相关的所有类

    * Context

      Context 模块构建于 Core Beans 模块基础之上，提供了一种类似于刑DI 注册器的框 架式的对象访问方法

    * Expression Language

      Expression Language 模块提供了强大的表达式语言，用于在运行时查询和操纵对象

  * Data Access/Integration

    * JDBC
    * ORM
    * OXM
    * JMS
    * Transaction

  * Web

    * Web模块
    * Web-Servlet模块
    * Web-Struts（已弃用
    * Web-Porlet模块

  * AOP

    AOP 模块提供了一个符合 AO 联盟标准的面向切面编程的实现，它让你可以定义例如方 法拦截器和切点，从而将逻辑代码分开，降低它们之间的调合性 利用 source-level 的元数据 功能，还可以将各种行为信息合并到你的代码中

  * Test

    JUnit及TestNG

### 2.容器的基本实现

* Spring结构组成

  * DefaultlistableBeanFactory

    XmlBeanFactory继承向DefaultListableBeanFactory，而DefaultListableBeanFactory 是整个 bean 加载的核心部分，是Spring注册及加载Bean的默认实现，而对于 XmlBeanFactory与DefaultListableBeanFactory 不同的地方其实是在 XmlBeanFactory 中使用了自定义的 XML 读取器 XmlBeanDefinitionReader ，实现了个性化的 BeanDefinitionReader读取， DefaultListableBeanFactory 继承了 AbstractAutowireCapableBeanFactory 并实现了ConfigurableListableBeanFactory以及BeanDefinitionRegistry接口

  * XmlBeanDefinitionReader

* XmlBeanFactory

  * 配置文件封装
  * 加载Bean

* 获取xml的验证模式

* 加载xml文件，并得到对应的Document

* 解析及注册BeanDefinitions

### 3.默认标签的解析

* 对bean标签的解析及注册
  * 解析BeanDefinition
    1. 创建用于属性承载的BeanDefinition
    2. 解析各种属性
    3. 解析子元素meta
    4. 解析子元素lookup-method
    5. 解析子元素replaced-method