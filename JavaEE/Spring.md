# 1.Spring概述
IoC 反转控制 AOP面向切片编程为内核
Spring优势：方便解耦，简化开发，AOP变成的支持，声明式事务的支持，方便程序的测试，方便继承各种优秀的框架，降低java EE API的使用难度，java源码使经典学习范例
Spring体系结构	<img src="https://img-blog.csdnimg.cn/20191216224813110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjgyNTI1,size_10,color_FFFFFF,t_70" alt="Spring Framework Runtime" style="zoom: 50%;" />
# 2. IoC的概念
### 程序的耦合和解耦
- 开发中应注意编译期不依赖，运行时才依赖
- 解耦思路
使用反射来创建对象，避免使用new关键字
通过读取配置文件来获取要创建的对象全限定类名
### 工厂模式解耦
- 创建Bean对象的工厂，Bean指可重用的组件
### 控制反转IoC
# 3.使用Spring的IoC解决程序的耦合
