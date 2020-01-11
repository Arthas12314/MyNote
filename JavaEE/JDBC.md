# JDBC
- 概念 Java DataBase Connectivity java数据库连接/java操作数据库，定义了操作所有关系型数据库的规则（接口）
- 简单案例

```
//导入jar包，注册驱动
Class.forName("com.mysql.jdbc.Driver");
//获取数据库连接
Connection conn=DriverManager.getConnection("jdbc:mysql://localhost:3306/db3","root","root");
//定义sql语句
String sql="udpate account set balance=500 where id=1";
//获取执行sql的对象
statement stmt=conn.createstatement();
//执行sql
int count=stmt.executeUpdate(sql);
//处理结果
System.out.println(count);
//释放资源
stmt.close();
conn.close();
```
### 详解各个对象：
- DriverManager:驱动管理对象
	* 功能
		注册驱动 registerDriver 
		获取数据库连接
	* 参数： url指定连接的路径jdbc:mysql://ip:端口号/数据库名称
	* 细节：如果连接的是本机mysql服务器，且服务端口默认为3306，可简写为 jdbc:mysql:///数据库名称
- connection：数据库连接对象
	* 方法 Statement createStatement() PreparedStatement preparedStatement(String sql)
	* 管理事务：
		+ 开启事务：setAutoCommited(boolean autoCommit)设置false则开启事务
		+ 提交事务：commit()
		+ 回滚事务：rollback()
- Statement：执行sql的对象
	* 方法
		+ boolean execute(String sql):可以执行任意的语句
		+ int executeUpdate(String sql):执行DML(insert update delete)语句，返回影响行数
		+ ResultSet executeQuery(String sql)：执行DQL(select)语句
- ResultSet：结果集对象
- PreparedStatement:功能增加的执行sql的对象