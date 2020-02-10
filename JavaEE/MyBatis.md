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

    ```xml
    <!-- 配置环境 -->
        <environments default="mysql">
            <!-- 配置mysql的环境-->
            <environment id="mysql">
                <!-- 配置事务的类型-->
                <transactionManager type="JDBC"/>
                <!-- 配置数据源（连接池） -->
                <dataSource type="POOLED">
                    <!-- 配置连接数据库的4个基本信息 -->
                    <property name="driver" value="com.mysql.jdbc.Driver"/>
                    <property name="url" value="jdbc:mysql://localhost:3306/eesy"/>
                    <property name="username" value="root"/>
                    <property name="password" value="1234"/>
                </dataSource>
            </environment>
        </environments>
    ```

  * 创建映射配置文件

    ```xml
    <mapper namespace="com.itheima.dao.IUserDao">
        <!--配置查询所有-->
        <select id="findAll" resultType="com.itheima.domain.User">
            select * from user
        </select>
    </mapper>
    ```

  * 环境搭建注意事项
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

* 自定义Mybatis的设计模式

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
  ```

  * 工厂模式

  * 构造者模式

    在创建工厂时使用SqlSessionFactoryBuilder进行构建

  * 代理模式

* 自定义Mybatis的分析

  1. 根据配置文件的信息创建Connection对象

     注册驱动获取链接

  2. 获取预处理对象PreparedStatement

     conn.preparedStatement(sql);

  3. 执行查询

     ```ResultSet resultSet=preparedStatement.executeQuery();```

  4. 遍历结果进行封装

     ```java
     List list=new ArrayList<>();
     while(resultSet.hasNext()){
     	E element=(E)Class.forName(配置的全限定类名).newInstance();
         /**
         * 进行封装，把每个rs的内容都添加到element中
        	* 我们就把表的列名看成实体类的属性名称
        	* 就可以使用反射的方式来根据名称获得每个属性，并把值赋进去
        	*/
         list.add(element);
     }
     return(list);
     ```

  * 使以上方法执行需要提供两个信息

    * 连接信息

    * 映射信息

      映射信息包含执行的sql以及封装的实体类的全限定类名，二者合并为一个对象Mapper

  * ```java
    IUserDao userDao = session.getMapper(IUserDao.class);
    
    //根据Dao接口的字节码创建dao的代理对象
    
    public <T> T getMapper(Class<T> daoInterface){
        /**
        * 类加载器：使用和被代理对象相同的类加载器
        * 代理对象实现的接口：和被代理对象相同的接口
        * 如何被代理：即InvocationHandler接口的自定义实现类，并实现selectList方法
        */
    	Proxy.newProxyInstance(类加载器，代理对象要实现的接口字节码，如何代理);
    }
    ```

## 3.MyBatisCRUD

* M'y'Batis自定义流程

  * SqlSessionFactoryBuilder接收SqlMapConfig.xml文件流，构建出SqlSessionFactory对象

  * SqlSessionFactory读取SqlMapConfig.xml中连接数据库信息和mapper映射信息，用来生产出真正操作数据库的SqlSession对象

  * SqlSession作用  无论哪个分支，除了获取数据库信息，还需要得到sql语句

    * 生成代理接口  

      SqlSessionImpl对象的getMapper方法分两步来实现

      1. 先用SqlSessionFactory读取的数据库连接信息建立Connection对象
      2. 通过jdk代理模式创建出代理对象作为getMapper的方法返回值，这里主要是在创建代理对象时第三个参数处理类里面得到sql语句执行对应的CRUD操作

    * 定义通用CRUD方法

      SqlSessionImpl对象中提供selectList()方法

      1. 用SqlSessionFactory读取的数据库连接信息创建出jdbc的Connection对象
      2. 直接得到sql语句，使用jdbc的Connection对象进行对应的CRUD操作

  * 封装结果集

    将结果封装为java对象返回调用者，因此需要获取返回的结果类型

* CRUD

  * 查询
  
  ```xml
      <!--根据id查询-->
    <select id="findById" resultType="com.itheima.domain.User" parameterType="int">
      	select * from user where id = #{uid}
    </select>
  ```

    resultType属性：用于指定结果集的类型。
  
    parameterType属性：用于指定传入参数的类型。
  
    sql语句中使用#{}字符：它代表占位符，相当于jdbc部分的?，都是用于执行语句时替换实际的数据。具体的数据是由#{}里面的内容决定的。
  
    #{}中内容的写法：由于数据类型是基本类型，所以此处可以随意写。
  
  * 插入
  
    ```xml
    <!--保存用户-->
    <insertid="saveUser" parameterType="com.itheima.domain.User">
    	insert into user(username,birthday,sex,address) 
    		values(#{username},#{birthday},#{sex},#{address})
    </insert>
    ```
  
    parameterType属性：代表参数的类型，因为我们要传入的是一个类的对象，所以类型就写类的全名称。sql语句中使用#{}字符：它代表占位符，相当于jdbc部分的?，都是用于执行语句时替换实际的数据。具体的数据是由#{}里面的内容决定的。
  
    #{}中内容的写法：由于我们保存方法的参数是一个User对象，此处要写User对象中的属性名称。它用的是ognl表达式。
  
    ognl表达式：它是apache提供的一种表达式语言，全称是：Object Graphic Navigation Language  对象图导航语言它是按照一定的语法格式来获取数据的。语法格式就是使用#{对象.对象}的方式，\#{user.username}它会先去找user对象，然后在user对象中找到username属性，并调用getUsername()方法把值取出来。但是我们在parameterType属性上指定了实体类名称，所以可以省略user.而直接写username
  
    ```xml
    <!-- 新增用户后，同时还要返回当前新增用户的id值，因为id是由数据库的自动增长来实现的，所以就相当于我们要在新增后将自动增长auto_increment的值返回。 -->
    <insertid="saveUser" parameterType="USER">
        <!--配置保存时获取插入的id -->
        <selectKey keyColumn="id" keyProperty="id" resultType="int">
            select last_insert_id();
        </selectKey>
        insert into user(username,birthday,sex,address) values(#{username},#{birthday},#{sex},#{address})
    </insert>
    ```
  
  * 更新用户
  
    ```xml
    <!--更新用户-->
    <updateid="updateUser" parameterType="com.itheima.domain.User">
    	update user set username=#{username},birthday=#{birthday},sex=#{sex},address=#{address} where id=#{id}
    </update>
    ```
  
  * 删除用户
  
    ```xml
    <!--删除用户-->
    <deleteid="deleteUser" parameterType="java.lang.Integer">
    	delete from user where id = #{uid}
    </delete>
    ```
  
  * 模糊查询
  
    ```xml
    <!--根据名称模糊查询--><selectid="findByName" resultType="com.itheima.domain.User" parameterType="String">
    	select * from user where usernamelike #{username}
    </select>
    ```
  
    * \#{}表示一个占位符号
  
      通过#{}可以实现preparedStatement向占位符中设置值，自动进行java类型和jdbc类型转换，#{}可以有效防止sql注入。#{}可以接收简单类型值或pojo属性值。如果parameterType传输单个简单类型值，#{}括号中可以是value或其它名称。
  
    * ${}表示拼接sql串
  
      通过${}可以将parameterType 传入的内容拼接在sql中且不进行jdbc类型转换，${}可以接收简单类型值或pojo属性值，如果parameterType传输单个简单类型值，${}括号中只能是value。
  
* SqlMapConfig.xml

  * -properties（属性）

    --property

  * -settings（全局配置参数）

    --setting

  * -typeAliases（类型别名）

    --typeAliase

    --package

  * -typeHandlers（类型处理器）

  * -objectFactory（对象工厂）

  * -plugins（插件）

  * -environments（环境集合属性对象）

    --environment（环境子属性对象）

    * ---transactionManager（事务管理）
    * ---dataSource（数据源）

  * -mappers（映射器）

    * --mapper
    * --package

  ```xml-dtd
  <configuration>
      <!-- 配置properties
          可以在标签内部配置连接数据库的信息。也可以通过属性引用外部配置文件信息
          resource属性： 常用的
              用于指定配置文件的位置，是按照类路径的写法来写，并且必须存在于类路径下。
          url属性：
              是要求按照Url的写法来写地址
              URL：Uniform Resource Locator 统一资源定位符。它是可以唯一标识一个资源的位置。
              它的写法：
                  http://localhost:8080/mybatisserver/demo1Servlet
                  协议      主机     端口       URI
  
              URI:Uniform Resource Identifier 统一资源标识符。它是在应用中可以唯一定位一个资源的。
  
      -->
      <properties url="file:///C:/Users/hawk4/IdeaProjects/MybatisLearning/Mybatis_day02/day02_eesy_01mybatisCRUD/src/main/resources/jdbcConfig.properties">
      </properties>
  
      <!--使用typeAliases配置别名，它只能配置domain中类的别名 -->
      <typeAliases>
          <!--typeAlias用于配置别名。type属性指定的是实体类全限定类名。alias属性指定别名，当指定了别名就再区分大小写 
          <typeAlias type="com.itheima.domain.User" alias="user"></typeAlias>-->
  
          <!-- 用于指定要配置别名的包，当指定之后，该包下的实体类都会注册别名，并且类名就是别名，不再区分大小写-->
          <package name="com.itheima.domain"/>
      </typeAliases>
  
      <!--配置环境-->
      <environments default="mysql">
          <!-- 配置mysql的环境-->
          <environment id="mysql">
              <!-- 配置事务 -->
              <transactionManager type="JDBC"/>
  
              <!--配置连接池-->
              <dataSource type="POOLED">
                  <property name="driver" value="${jdbc.driver}"/>
                  <property name="url" value="${jdbc.url}"/>
                  <property name="username" value="${jdbc.username}"/>
                  <property name="password" value="${jdbc.password}"/>
                  <!--<property name="driver" value="com.mysql.jdbc.Driver"/>
                  <property name="url" value="jdbc:mysql://localhost:3306/eesy"/>
                  <property name="username" value="root"/>
                  <property name="password" value="1234"/>-->
              </dataSource>
          </environment>
      </environments>
      <!-- 配置映射文件的位置 -->
      <mappers>
          <!--<mapper resource="com/itheima/dao/IUserDao.xml"></mapper>-->
          <!-- package标签是用于指定dao接口所在的包,当指定了之后就不需要在写mapper以及resource或者class了 -->
          <package name="com.itheima.dao"/>
      </mappers>
  </configuration>
  ```

* mybatis中的连接池

  可以减少我们获取连接所消耗的时间

  mybatis连接池提供了3种方式的配置：

  * 配置的位置：
    主配置文件SqlMapConfig.xml中的dataSource标签，type属性就是表示采用何种连接池方式。

  * type属性的取值：

    * POOLED

      采用传统的javax.sql.DataSource规范中的连接池，mybatis中有针对规范的实现

    * UNPOOLED

      采用传统的获取连接的方式，虽然也实现Javax.sql.DataSource接口，但是并没有使用池的思想。

    * JNDI

      采用服务器提供的JNDI技术实现，来获取DataSource对象，不同的服务器所能拿到DataSource是不一样。注意：如果不是web或者maven的war工程，是不能使用的。

* 动态sql

  * \<if\>

  * \<where\>

    ```xml
    <select id="findUserByCondition" resultMap="userMap" parameterType="user">
            select * from user
            <where>
                <if test="userName != null">
                    and username = #{userName}
                </if>
                <if test="userSex != null">
                    and sex = #{userSex}
                </if>
            </where>
        </select>
    ```

  * \<foreach\>

    ```xaml
    <!-- 根据queryvo中的Id集合实现查询用户列表 -->
        <select id="findUserInIds" resultMap="userMap" parameterType="queryvo">
            <include refid="defaultUser"/>
            <where>
                <if test="ids != null and ids.size()>0">
                    <foreach collection="ids" open="and id in (" close=")" item="uid" separator=",">
                        #{uid}
                    </foreach>
                </if>
            </where>
        </select>
    ```

* 多表查询

  * 一对一查询（多对一

    ```xml
    <!-- 定义封装account和user的resultMap -->
        <resultMap id="accountUserMap" type="account">
            <id property="id" column="aid"/>
            <result property="uid" column="uid"/>
            <result property="money" column="money"/>
            <!-- 一对一的关系映射：配置封装user的内容-->
            <association property="user" column="uid" javaType="user">
                <id property="id" column="id"/>
                <result column="username" property="username"/>
                <result column="address" property="address"/>
                <result column="sex" property="sex"/>
                <result column="birthday" property="birthday"/>
            </association>
        </resultMap>
        
    	<!--查询所有账户同时包含用户名和地址信息-->
        <select id="findAllAccount" resultType="accountuser">
            select a.*,u.username,u.address from account a , user u where u.id = a.uid;
        </select>
    ```

  * 一对多查询

    ```xml
    <!-- 定义User的resultMap-->
        <resultMap id="userAccountMap" type="user">
            <id property="id" column="id"/>
            <result property="username" column="username"/>
            <result property="address" column="address"/>
            <result property="sex" column="sex"/>
            <result property="birthday" column="birthday"/>
            <!-- 配置user对象中accounts集合的映射 -->
            <collection property="accounts" ofType="account">
                <id column="aid" property="id"/>
                <result column="uid" property="uid"/>
                <result column="money" property="money"/>
            </collection>
        </resultMap>
        
        <!-- 查询所有 -->
        <select id="findAll" resultMap="userAccountMap">
            select * from user u left outer join account a on u.id = a.uid
        </select>
    ```

  * 多对多查询

    ```xml
    	<!-- 定义User的resultMap-->
        <resultMap id="userMap" type="user">
            <id property="id" column="id"/>
            <result property="username" column="username"/>
            <result property="address" column="address"/>
            <result property="sex" column="sex"/>
            <result property="birthday" column="birthday"/>
            <!-- 配置角色集合的映射 -->
            <collection property="roles" ofType="role">
                <id property="roleId" column="rid"/>
                <result property="roleName" column="role_name"/>
                <result property="roleDesc" column="role_desc"/>
            </collection>
        </resultMap>
    
        <!-- 查询所有 -->
        <select id="findAll" resultMap="userMap">
            select u.*,r.id as rid,r.role_name,r.role_desc from user u
             left outer join user_role ur  on u.id = ur.uid
             left outer join role r on r.id = ur.rid
        </select>
    ```

* 延迟加载策略

  * 多对一

    ```xml
    <!-- 定义封装account和user的resultMap -->
        <resultMap id="accountUserMap" type="account">
            <id property="id" column="id"/>
            <result property="uid" column="uid"/>
            <result property="money" column="money"/>
            <!-- 一对一的关系映射：配置封装user的内容
            select属性指定的内容：查询用户的唯一标识：
            column属性指定的内容：用户根据id查询时，所需要的参数的值
            -->
            <association property="user" column="uid" javaType="user" select="com.itheima.dao.IUserDao.findById"/>
        </resultMap>
    
    	<!-- 查询所有 -->
        <select id="findAll" resultMap="accountUserMap">
            select * from account
        </select>
    ```

  * 一对多

    ```xml
    <!-- 定义User的resultMap-->
        <resultMap id="userAccountMap" type="user">
            <id property="id" column="id"/>
            <result property="username" column="username"/>
            <result property="address" column="address"/>
            <result property="sex" column="sex"/>
            <result property="birthday" column="birthday"/>
            <!-- 配置user对象中accounts集合的映射 -->
            <collection property="accounts" ofType="account" select="com.itheima.dao.IAccountDao.findAccountByUid" column="id"/>
        </resultMap>
    
        <!-- 查询所有 -->
        <select id="findAll" resultMap="userAccountMap">
            select * from user
        </select>
    ```

* Mybatis缓存

  * 缓存是存在于内存中的临时数据。
    减少和数据库的交互次数，提高执行效率。

    * 适用于缓存：
      经常查询并且不经常改变的。
      数据的正确与否对最终结果影响不大的。
    * 不适用于缓存：
      经常改变的数据
      数据的正确与否对最终结果影响很大的。例如：商品的库存，银行的汇率，股市的牌价。

  * Mybatis中的一级缓存和二级缓存

    * 一级缓存：
      Mybatis中SqlSession对象的缓存。
      当我们执行查询之后，查询的结果会同时存入到SqlSession为我们提供一块区域中。
      该区域的结构是一个Map。当我们再次查询同样的数据，mybatis会先去sqlsession中
      查询是否有，有的话直接拿出来用。
      当SqlSession对象消失时，mybatis的一级缓存也就消失了。

    * 二级缓存:
      Mybatis中SqlSessionFactory对象的缓存。由同一个SqlSessionFactory对象创建的SqlSession共享其缓存。
      二级缓存的使用步骤：

      1. 让Mybatis框架支持二级缓存（在SqlMapConfig.xml中配置）

         ```xml
         <settings>
         	<setting name="cacheEnabled" value="true"/>
         </settings>
         ```

      2. 让当前的映射文件支持二级缓存（在IUserDao.xml中配置）

         ```xml
         <!--开启user支持二级缓存-->
         <cache/>
         ```

      3. 让当前的操作支持二级缓存（在select标签中配置）

         ```xml
         <!-- 根据id查询用户 -->
         <select id="findById" parameterType="INT" resultType="user" useCache="true">
             select * from user where id = #{uid}
         </select>
         ```

* 使用注解开发

  * 一对一、多对一

    ```java
    /**
    * 查询所有账户，并且获取每个账户所属的用户信息
    * @return
    */
        @Select("select * from account")
        @Results(id="accountMap",value = {
            @Result(id=true,column = "id",property = "id"),
            @Result(column = "uid",property = "uid"),
            @Result(column = "money",property = "money"),
            @Result(property = "user",column = "uid", one=@One(select="com.itheima.dao.IUserDao.findById",fetchType= FetchType.EAGER))
        })
    List<Account> findAll();
    
    /**
    * 根据用户id查询账户信息
    * @param userId
    * @return
    */
    @Select("select * from account where uid = #{userId}")
    List<Account> findAccountByUid(Integer userId);
    ```

  * 一对多

    ```java
    /**
    * 查询所有用户
    * @return
    */
    @Select("select * from user")
        @Results(id="userMap",value={
            @Result(id=true,column = "id",property = "userId"),
            @Result(column = "username",property = "userName"),
            @Result(column = "address",property = "userAddress"),
            @Result(column = "sex",property = "userSex"),
            @Result(column = "birthday",property = "userBirthday"),
            @Result(property = "accounts",column = "id",
                many = @Many(select = "com.itheima.dao.IAccountDao.findAccountByUid",
                	fetchType = FetchType.LAZY))
        })
    List<User> findAll();
    
    /**
    * 根据id查询用户
    * @param userId
    * @return
    */
    @Select("select * from user  where id=#{id} ")
    @ResultMap("userMap")
    User findById(Integer userId);
    
     /**
     * 根据用户名称模糊查询
     * @param username
     * @return
     */
     @Select("select * from user where username like #{username} ")
     @ResultMap("userMap")
     List<User> findUserByName(String username);
    ```

  * 缓存配置

    ```java
    @CacheNamespace(blocking = true)
    ```
