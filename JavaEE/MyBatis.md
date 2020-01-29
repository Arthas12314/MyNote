# MyBatis

## 1.概述

* mybatis 是一个优秀的基于 java 的持久层框架，它内部封装了 jdbc，使开发者只需要关注 sql 语句本身， 而不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。 

* mybatis 通过 xml 或注解的方式将要执行的各种 statement 配置起来，并通过 java 对象和 statement 中 sql 的动态参数进行映射生成最终执行的 sql 语句，最后由 mybatis 框架执行 sql 并将结果映射为 java 对象并 返回。 

* 采用 ORM 思想解决了实体和数据库映射的问题，对 jdbc 进行了封装，屏蔽了 jdbc api 底层访问细节，使我 们不用与 jdbc api 打交道，就可以完成对数据库的持久化操作。

* ORM（Object Relational Mappging）对象关系映射

  即把数据库表和实体类即实体类的属性对应起来，让我们可以操作实体类实现操作数据库表

## 2.入门

* 环境搭建

  * 创建maven工程并导入坐标
  * 创建实体类和dao的接口
  * 创建Mybatis的主配置文件
  * 创建映射配置文件
  * 环境搭建注意事项
    * 创建 UserDao.xml 和 UserDao.java 是为了和我们之前的知识保持一致。
    * 在mybatis中它把持久层的操作接口名称和映射文件也叫做：Mapper
    * mybatis的映射配置文件配置必须和dao接口的包结构相同。
    * 映射配置文件的Mapper标签和namesplace的属性值必须是dao接口的全限定类名。
    * 映射配置文件的操作配置(select)，id属性的取值必须是dao类接口。
    * 当我们遵从了以上注意项之后，我们在开发中就无需再写dao的实现类。

* 入门案例

  ```java
  //1.读取配置文件
  InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
  //2.创建SqlSessionFactory工厂
  SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
  SqlSessionFactory factory = builder.build(in);
  //3.使用工厂生产SqlSession对象
  SqlSession session = factory.openSession();
  //4.使用SqlSession创建Dao接口的代理对象
  IUserDao userDao = session.getMapper(IUserDao.class);
  //5.使用代理对象执行方法
  List<User> users = userDao.findAll();
  for(User user : users){
  System.out.println(user);
  }
  //6.释放资源
  session.close();
  in.close();
  ```

  注意事项：

  不要忘记在映射配置中告知Mybatis要封装到哪个实体类中，配置的方式:指定实体类的全限定类名

* Mybatis基于注解的入门案例：

  * 把IUserDao移除，在dao接口的方法上使用@Select注解，并且指定SQL语句，同时需要在SqlMapConfig.xml的Mapper配置时，使用class属性指定dao接口的全限定类名
  * 实际开发中一般不写dao实现类（无论是xml还是注解，而Mybatis也支持写dao实现类

* 