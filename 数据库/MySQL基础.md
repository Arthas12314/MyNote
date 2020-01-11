# 数据库

* 概念：
  * 数据是数据库中储存的基本对象，数据的含义为数据的语义
  * 数据库DB：长期储存、有组织、可共享、大量数据：长期储存、有组织、可共享、大量数据
    * 按一定的数据模型组织、描述和储存；可共享；冗余度较小；数据独立性较高；易扩展
* 数据库管理系统DBMS
  * 数据定义，数据组织、储存和管理，数据操纵
  * 主要功能：运行管理、建立和维护
* DBMS特点
  * 数据放到表中，表再放在库中
  * 一个数据库中可以有多个表，每个表都有一个名字标识，具有唯一性
  * 表具有一些特性，定义了数据再表中如何储存，类似java中“类”
  * 表由列组成，也称字段，所有表都是由一个或多个列组成的，每一列类似java中的”属性“
  * 表中的数据是按行储存的，每一行类似java中的“对象”

* Mysql启动： mysql -h 主机名 -P端口号 -u 用户名 -p 密码

```
show databases;
use 库名;
show tables;
show tables from 库名;
create table 表名(
	列名 列类型,
	列名 列类型,
	...
);
```

* 查看表结构： desc 表名
* mysql不区分大小写，但关键字大写，表明列名小写

## 1.DQL语言

做查询时前面添加命令 USE 库名;

查询时，先确认是否所需数据都在一张表里，决定是否连接表，再进行筛选，注意GROUP BY前后筛选

### 基础查询

* select 查询列表 from 表名

* 查询列表可以是：表中的字段、常量值、表达式、函数

* 查询的结果是一个虚拟的表格

  ```mysql
  -- 查询表中的单个字段
  SELECT last_name FROM employees;
  -- 查询表中的多个字段
  SELECT last_name,salary,email FROM employees;
  -- 查询表中的所有字段
  SELECT * FROM employees;-- 星号不可以自定义顺序
  -- 查询常量值
  SELECT 100; SELECT 'john';
  -- 查询表达式
  SELECT 100%98；
  -- 查询函数
  SELECT version();
  -- 可以起别名 别名可加双引号避免歧义报错
  SELECT 100%98 AS 结果;
  SELECT last_name AS 姓,first_name AS 名 FROM employees;
  SELECT last_name 姓,first_name 名 FROM employees;
  -- 去重
  SELECT DISTINCT department_id FROM employees;
  -- 加号
  -- mysql中对字符型进行转换，转换成功则继续运算，否则字符型转化为0，如果有一方为null，则结果输出null
  SELECT CONCAT('a','b','c') AS 结果; 则输出 abc
  IFNULL(字段,数值)
  ```

### 条件查询

```
SELECT 查询列表
FROM 表名
WHERE 筛选条件
```

* 支持条件运算符（包括模糊查询）、逻辑表达式AND NOT OR

  * 模糊查询：like、between and、in、is null

    * like

      通配符：%->0-n个任意字符、一个任意字符用\\转义，
      
      或 ```last_name LIKE '_$_%' ESCAPE '$';($可换成任意字符充当转义)```
      
    * between and 包含为闭区间 
      
    * in job_id('','','');
      
    * is null/is not null ->由于= <> 不可判断null值
      
    * <=>安全等于（可以判断null

### 排序查询

```mysql
SELECT 查询列表 
FROM 表 
where 筛选条件 
ORDER BY 排序列表(ASC|DESC);
```

* 特点：默认升序；

  order by 支持单个多个字段、表达式、函数、别名；一般放在查询语句最后（只有Limit 子句在其后

  ```mysql
  SELECT * FROM employees ORDER BY salary DESC;
  SELECT * FROM employees ORDER BY salary ASC;
  
  -- 按年薪高低显示员工的信息和 年薪按表达式顺序
  SELECT *,salary*12*(1+IFNULL(commission_pct,0)) 年薪 FROM employees 
  ORDER BY salary*12*(1+IFNULL(commission_pct,0)) DESC;
  
  -- 按年薪高低显示员工的信息和 年薪按别名顺序
  SELECT *,salary*12*(1+IFNULL(commission_pct,0)) 年薪
  FROM employees 
  ORDER BY 年薪 DESC;
  
  -- 按函数排序
  SELECT LENGTH(last_name) 字节长度，last_name,salary FROM employees 
  ORDER BY LENGTH(last_name) DESC;
  
  -- 按多个字段排序
  SELECT * FROM employees 
  ORDER BY salary ASC,employees_id DESC;
  ```

### 常见函数

类似于方法

* 单行函数：concat、length、ifnull等

  * 字符函数

    * SELECT LENGTH 注意这里是字节长度
    * CONCAT( , , )
    * UPPER,LOWER
    * SUBSTR、SUBSTRING sql中索引从1开始，substr的第三个参数为长度类似C++substr
    * INSTR 返回子串中出现的第一次起始位置，否则返回0
    * TRIM （'a' FROM 'aaaaneawwwaaaa'）则仅去掉前后两端的a
    * LPAD ('test',10,'*') output 则左边用星号填充至10个字符（如果输出长度小于原字符则截断
    * RPAD同理
    * REPLACE('test','e','s')

  * 数学函数

    * ROUND四舍五入 
    * SELECT ROUND(1.45);
    * CEIL向上取整
    * FLOOR返回<=该参数的最大整数
    * TRUNCATE截断 SELECT TRUNCATE(1.555，1)保留一位小数
    * MOD取余

  * 日期函数

    * SELECT NOW();返回当前系统日期+时间
    * SELECT CURDATE();返回当前系统日期
    * SELECT CURTIME();返回当前时间
    * SELECT year(),month(),day(),hour(),minute(),second() 获取年月日时分秒
    * SELECT MONTHNAME(NOW());
    * str_to_date将日期格式的字符串转汉城指定格式的日期  STR_TO_DATE('9-13-1999','%m-%d=%Y');
    * date_format:将日期转换成字符
    * SELECT DATE_FORMAT(NOW(),'%y年%m月%d日') AS out_put;

  * 其他函数

    * SELECT VERSION();					
    * SELECT DATABASE();				
    * SELECT USER();
    * SELECT PASSWORD('');//字符加密

  * 流程控制函数
      	
       * IF类似三目运算符?:
       
       * SELECT IF(10>5,'大'，'小');
       
       * CASE
       
         ```mysql
         -- 类似switch
         CASE department_id
         WHEN 30 THEN salary*1.1
         WHEN 40 THEN salary*1.2
         ELSE salary
         END AS 新工资
         FROM employees;
         -- 类似多重if-else
         CASE
         WHEN salary>20000 THEN 'A'
         WHEN salary>15000 THEN 'B'
         ELSE 'D'
         END AS 工资级别
         FROM employees;
         ```
  
  * 分组函数：
  
    用于统计又称聚合、统计函数，SUM、AVG、MAX、MIN、COUNT
  
    * SUM、AVG对数值型处理
  
    * MAX、MIN、COUNT对任何类型可处理
  
    * 所有的分组函数都忽略NULL值；都可以和DISTINCT搭配实现去重
  
    * COUNT详解
  
      ```mysql
      -- 统计行数
      SELECT COUNT(*) FROM employees;
      SELECT COUNT(1) FROM employees;
      ```
  
      MYISAM引擎COUNT(*)效率高INNODB两者差不多，比COUNT(字段)效率高
      和分组函数一同查询的字段要求是group by后的字段
      DATEDIFF 计算两参数差值

### 分组查询

```
SELECT 分组函数，列
FROM 表
WHERE 删选条件
GROUP BY 分组的列表
ORDER BY 子句
```

* 案例

  ```mysql
  SELECT AVG(salary)，department_id
  FROM employees
  WHERE email LIKE '%a%'
  GROUP BY department_id;
  				
  SELECT COUNT(*),department_id
  FROM employees
  GROUP BY department_id
  HAVING COUNT(*)>2;
  				
  SELECT MAX(salary),job_id
  FROM employees
  WHERE commission_pct IS NOT NULL
  GROUP BY job_id
  HAVING MAX(salary)>12000
  				
  SELECT MIN(salary),manager_id
  FROM employees
  WHERE department_id>102
  GROUP BY manager_id
  HAVING MIN(salary)>5000;
  ```

* 特点

  筛选条件 两者筛选数据源不同 

  分组前筛选 筛选的是原始表

  分组后筛选 筛选的是分组后的结果集

  能分组前筛选的尽量分组前筛选（性能考虑

* GROUP BY
  
  支持多分组 支持按表达式或函数分组
  
  ```mysql
  -- 按员工姓名的长度分组，查询每一组的员工个数，筛选员工个数>5的有哪些
  SELECT COUNT(*),LENGTH(last_name) len_name
  FROM employees
  GROUP BY LENGTH(last_name)
  HAVING COUNT(*)>5;
  ```

### 连接查询

又称多表查询

```mysql
SELECT 查询列表
FROM 表1 别名 连接类型
JOIN 表2 别名
ON 连接条件
```

* 内连接 INNER JOIN

  * 等值连接

    ```mysql
    SELECT NAME,boyName FROM boys,beauty
    WHERE beauty.boyfriend_id=boys.id;
    ```

    * 查询员工名、工种号、工种名 可以为表起别名

      ```mysql
      SELECT last_name,employees.job_id,job_title
      FROM employees,jobs
      WHERE employees.`job_id`=jobs.`job_id`;
      ```

    * 可以加筛选
  
      ```mysql
      SELECT last_name,department_name						
      FROM employees,departments
      WHERE employees.`department_id`=departments.department_id
      AND employees.`commission_pct` IS NOT NULL;
      ```
  
    * 可以加分组
  
      ```mysql
      -- 查询有奖金的每个部门的部门名和部门的领导编号和该部门的最低工资
      SELECT department_name,d.manager_id,MIN(e.salary)
      FROM employees e,departments d
      WHERE e.department_id =d.department_id
      and e.commission_pct is not null
      GROUP BY department_name,d.manager_id;
      ```
  
    * 可以加排序
  
      ```mysql
      -- 查询每个工种的工种名和员工的个数,并按员工个数降序
      SELECT
          job_title,
          count(*)
      FROM
          employees e,
          jobs j
      WHERE
          e.job_id = j.job_id
      GROUP BY
          job_title
      ORDER BY
          count(*) DESC
      ```
  
    * 可以进行三表连接
  
      ```mysql
      SELECT last_name,department_name,city
      FROM employees e,departments d,locations l
      WHERE e.`department_id`=d.`department_id`
      AND d.`location_id`=l.`location_id`
      AND city LIKE 's%'
      ORDER BY department_name DESC;
      ```
  
    * 也可以加分组后筛选
  
  * 非等值连接
  
    ```mysql
    -- 查询员工的工资和工资级别
    SELECT salary,grade_level
    FROM employees e,job_grades g
    WHERE salary BETWEEN g.`lowest_sal` AND g.`highest_sal`;
    ```
  
  * 自连接
  
    ```mysql
    -- 查询员工名和上级的名称
    SELECT e.employee_id,e.last_name,m.employee_id,m.last_name
    FROM employees e,employees m
    WHERE e.`manager_id`=m.`employee_id`;
    ```
    
  * 注意
  
    * 多表连接的结果为多表的交集部分
    * n表连接至少需要n-1个连接条件
    * 多表的顺序无要求，一般需起别名
    * 可搭配其他子句使用
    
  * 总结
  
    ```mysql
    SELECT 查询列表
    FROM 表1 别名1，表2 别名2（等值连接、非等值连接 / FROM 表 别名1，表 别名2（自连接
    WHERE （非）等值的连接条件
    AND 筛选条件
    GROUP BY 分组字段
    HAVING 分组后的筛选
    ORDER BY 排序字段
    ```
  
* 外连接：连接类型outer

  用与查询一个表有，另一个表没有的记录

  * 左外连接

    ```mysql
    SELECT <select_list>
    FROM A
    LEFT OUTER JOIN B
    ON A.key=B.key
    WHERE B.key IS NULL;
    ```

    左外连接还返回左表中不符合连接条件单符合查询条件的数据行

  * 右外连接

    ```mysql
    SELECT <select_list>
    FROM B
    RIGHT OUTER JOIN A
    ON A.key=B.key
    WHERE A.key IS NULL;
    ```

    右外连接还返回右表中不符合连接条件单符合查询条件的数据行

  * 交叉连接

    ```mysql
    SELECT b.* ,bo.*
    FROM beauty b
    CROSS JOIN boys bo;
    -- 结果为笛卡尔乘积结果
    ```

### 子查询

> 出现在其他语句中的select语句，成为子查询或内查询

| 位置 | 子查询类型 |
| ---- | ---------- |
|select后|标量子查询|
|FROM后|表子查询|
|WHERE或HAVING后|标量子查询、列子查询、行子查询|
|EXISTS后|表子查询|

| 类型       | 特征                 |
| ---------- | -------------------- |
| 标量子查询 | 结果集只有一行一列   |
| 列子查询   | 结果集只有一行多列   |
| 行子查询   | 结果集有一行多列     |
| 表子查询   | 结果集一般为多行多列 |

* SELECT后面

  ```mysql
  -- 查询每个部门的员工个数
  SELECT d.*,(SELECT COUNT(*) 
  FROM employees 
  WHERE employees.`department_id`=d.`department_id`)
  FROM departments d
  ```

* FROM后面

  ```mysql
  -- 查询每个部门的平均工资的工资等级
  SELECT t1.*,t2.`grade_level`
  FROM(
      SELECT department_id,AVG(salary) avg_salary 
      FROM employees
      GROUP BY department_id
  ) t1 
  INNER JOIN job_grades t2
  ON t1.avg_salary BETWEEN t2.`lowest_sal` AND t2.`highest_sal`
  ```

* WHERE和HAVING后

  * 标量子查询	单行

    ```mysql
    -- 谁的工资比abel高
    SELECT e.`last_name` 
    FROM employees e 
    WHERE e.`salary`>(
        SELECT salary 
        FROM employees 
        WHERE last_name = 'Abel')；
    ```

  * 列子查询        多行
  
    ```mysql
    -- 返回location_id是1400或1700的部门中的所有员工姓名
    SELECT last_name
    FROM employees
    WHERE department_id IN(
        SELECT DISTINCT department_id
        FROM departments
        WHERE location_id IN(1400,1700)
    );
    -- 返回其他工种中比job_id为`IT_PROG`工种任一工资低的员工的员工号、姓名、job_id、salary
    SELECT last_name,employee_id,job_id,salary 
    FROM employees 
    WHERE salary<ALL( 
        SELECT DISTINCT salary 
        FROM employees 
        WHERE job_id = 'IT_PROG' ) 
    AND job_id<>'IT_PROG';
    ```
  
  * 行子查询         多列多行
  
    * 特点
  
      子查询放在小括号内、子查询一帮放在条件的右侧、标量子查询一般搭配着单行操作符使用
      列子查询，一般搭配着多行操作符使用
  
    ```mysql
    -- 员工编号最小并且工资最高的员工信息
    SELECT *
    FROM employees
    WHERE (employee_id,salary)=(
    	SELECT MIN(employee_id),MAX(salary)
    	FROM employees
    );
    -- 或用以下形式
    SELECT *
    FROM employees
    WHERE employee_id=(
    	SELECT MIN(employee_id)
    	FROM employees
    ) AND salary=(
    	SELECT MAX(salary)
    	FROM employees
    );
    ```
  
* EXISTS后面：相关子查询

  ```mysql
  SELECT EXISTS(
      SELECT employee_id 
      FROM employees  
      WHERE salary=30000
  );
  ```

### 分页查询

当显示的数据，一页显示不全，需要分页提交sql请求

```mysql
SELECT 查询列表
FROM 表
连接类型 JOIN 表2
ON 连接条件
WHERE 筛选条件
GROUP BY 分组字段
HAVING 分组后的筛选
ORDER BY 排序的字段
LIMIT offset,size;
-- 每页公式为 SELECT 查询列表 FROM 表 LIMIT (page-1)*size,size;
```

### union联合查询

将多条查询语句合并成一个结果

* 应用场景 	多个表没有连接关系，但是列数相同的表
* 特点   多条查询语句列数一致、字段名和类型顺序要一致、自动去重（取消去重则UNION ALL）

## 2.DML语言

### 插入语句insert

```mysql
-- 前者支持插入多行VALUES(),(),();后者不支持
-- 前者支持子查询 SELECT ,后者不支持
insert into 表名（列名，...）
values(值，...)
或
insert into 表名
set 列名=值,列名=值;
```

* 插入的类型要与列的类型一致或兼容

  ```mysql
  INSERT INTO beauty(id,NAME,sex,borndate,phone,photo,boyfriend_id)
  VALUES(13,'唐艺昕','女','1990-4-23','18988888888',NULL,2);
  ```

* 注意

  * 不可为NULL的值可以写NULL或省略
  * 可以省略列名，但是此时NULL值不能省略写

### 修改语句update

```mysql
-- 修改单表的记录
update 表名
set 列=新值,列=新值,...
where 筛选条件;
-- 修改多表的记录
update 表1
inner|left|right join 表2
set 列=值,列=值
where 筛选条件;

-- Example
UPDATE boys bo
INNER JOIN beauty b ON bo.`id`=b.`boyfriend_id`
SET b.`phone`=`114`
WHERE bo.`boyName`=`张无忌`;

UPDATE boys bo
RIGHT JOIN beauty b ON bo.`id`=b.`boyfriend_id`
SET b.`boyfriend_id`=2
WHERE bo.`id` IS NULL;
```

### 删除语句delete

```mysql
-- 单表删除
delete from 表1,表2
inner|left|right join 表2
on 连接条件
where 筛选条件;			
-- 多表删除
truncate table 表名;
```

* 注意
  * 如果删除的表中有自增长列，delete后再插入数据，值从断点开始，ertruncate从1开始
  * truncate 没有返回值，delete有返回值
  * truncate不能回滚，delete可以回滚

## 3.DDL语言

### 库的管理

* 创建

```mysql
create database 
IF NOT EXISTS 库名;
```
* 修改 (一般不做修改 

```mysql
-- 可改字符集：
ALTER DATABASE books 
CHARACTER SET gbk;
```
* 删除

```mysql
DROP DATABASE 
IF EXISTS books;
```

### 表的管理

* 创建

  ```mysql
  CREATE TABLE 表名(
  	列名 列的类型(长度)约束,
  	列名 列的类型(长度)约束,
  	列名 列的类型(长度)约束,
  );
  
  CREATE TABLE book(
  	id INT,#编号
  	bName VARCHAR(20),
  	authorId INT,
  	publishDate DATETIME
  );
  ```
```
  

* 修改

  ```mysql
  -- 修改列名
  ALTER TABLE book 
  CHANGE COLUMN publishdate pubDate DATETIME;
  -- 修改列的类型或约束
  ALTER TABLE book 
  MODIFY COLUMN pubdate TIMESTAMP;
  -- 添加新列
  ALTER TABLE author 
  ADD COLUMN annual DOUBLE;
  -- 删除列
  ALTER TABLE author 
  DROP COLUMN annual;
  -- 修改表名
  ALTER TABLE author 
  RENAME TO book_author;
```

* 删除

  ```mysql
  DROP TABLE 
  IF EXISTS book_author;
  -- 通用写法
  DROP DATABASE IF EXISTS 旧库名;
  CREATE DATABASE 新库名;
  DROP TABLE IF EXISTS 旧表名;
  CREATE TABLE 表名();
  ```

* 表的复制

  ```mysql
  -- 仅仅复制表的结构
  CREATE TABLE 
  LIKE author;
  复制表的结构+数据
  CREATE TABLE copy2
  SELECT * FROM author;
  -- 仅复制某些字段
  CREATE TABLE copy4
  SELECT id，au_name
  FROM author
  WHERE 0;
  ```

### 常见的数据类型

> 选择类型越简单越好，能保存数值的类型越小越好

* 整形

  * 用UNSIGNED标识有无符号，超范围则为临界值，如果不设置长度，会有默认的长度
  * 长度代表了显示的最大宽度，如果不够会用0在左边填充，但必须搭配ZEROFILL

* 小数

  * 浮点数  float(M,D) double(M,D)
  * 定点数  DEC(M,D) DECIMAL(M,D)
  * 特点：M为总位数，D为小数位数，默认float、double符合范围即可，decimal默认10,0	

* 字符型

  * 较短文本char、varchar

    char固定长度字符M默认为1，varchar可变长度字符M无默认必须指定，char性能高

  * 较长文本test、blob

* 枚举

* 集合

  ```
  CREATE TABLE tab_set(
  	s1 SET/ENUM('a','b','c','d')
  );
  			
  ```

  集合插入多个，枚举插入一个

* 日期型

  date、datetime、timestamp、time、year
  datetime从1000-1-1到9999-12-31
  timestamp则和实际时区有关，反应实际时间，且受mysql版本和sqlmode的影响，范围1970某时到2038某时

### 常见约束

限制表中的数据，为了保证表中的数据的准确性和可靠性

* 六大约束
  * NOT NULL
  * DEFAULT：保证该字段有默认值
  * PRIMARY KEY：主键，用于保证该字段的值的唯一性，并且非空
  * UNIQUE:唯一，用于保证该字段的值具有唯一性，可以为空
  * CHECK:（mysql中不支持）
  * FOREIGN KEY:外键，用于限制两个表的关系，用于保证该字段的值必须来自主表的关联列的值，在从表添加外键约束，用于引用主表中的某列的值，比如各种编号

* 添加约束时机

  * 创建表时

    * 列级约束
      直接在字段名和类型后面追加约束类型，六种语法都支持，但外键无效果

    * 表级约束
      六种中除了非空和默认都支持

      ```mysql
      [constraint 约束名] 约束类型(字段名) [FOREIGN KEY() REFERENCE 表名(字段名)];
      CREATE TABLE IF NOT EXISTS stuinfo(
          id INT PRIMARY KEY,
          stuname VARCHAR(20) NOT NULL,
          age INT DEFAULT 18,
          seat INT UNIQUE,
          majorid INT,
          CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCE major(id)
      );
      ```

    * 主键和UNIQUE：
      主键只能在一个表里有一个，但可以有联合主键（不推荐），UNIQUE可以有多个，也可以组合		

    * 外键
      要求在从表设置外键关系、从外的外键列的类型和朱标的关联列的类型要求一致或兼容，名称无要求
      主表的关联列必须是一个key（一般时主键或UNIQUE）
      删除数据时先删从表才能再删除主表

  * 修改表时

    * 列级约束

      ```mysql
      ALTER TABLE stuinfo 
      MODIFY COLUMN stuname VARCHAR(20) 约束;
      ```

    * 表级约束

      ```mysql
      ALTER TABLE stuinfo 
      ADD UNIQUE(seat);
      
      ALTER TABLE stuinfo 
      ADD FOREIGN KEY(majorid) REFERENCE major(id);
      ```

  * 修改表时删除约束

    ```mysql
    NOT NULL 改为NULL 默认改为不写 
    
    -- 删除主键
    ALTER TABLE stuinfo 
    DROP PRIMARY KEY;
    
    -- 删除UNIQUE 
    ALTER TABLE 表名 
    DROP INDEX 字段名;
    
    -- 删除外键 
    ALTER TABLE 表名 
    DROP FOREIGN KEY 字段名;
    ```

### 标识列：自增长列
* 类型后+AUTO_INCREMENT
* 要求仅给主键、外键或UNIQUE类型添加
* 一个表至多一个标识列
* 修改为非标识列时直接后面不写

## TCL语言

### 事务和事务处理

> 事务：一个或一组sql语句组成一个执行单元，这个执行单元要么全部执行，要么全部不执行

* ACID属性

  * 原子性：事务是一个不可分割的工作单位，操作要么都发生要么都不发生
  * 一致性：事务必须是一个数据库从一个一致性状态变换到另一个一致性状态
  * 隔离性：一个事务的执行不能被其他事务干扰，事务内部操作对其他并发事务隔离，不可相互干扰
  * 持久性：一个事务一旦被提交，他对数据库中的改变就是永久性的，后续操作不会有任何影响

* 隐式事务没有明显的开始和结束如：insert、update、delete

  一般的显式事务都是insert、update、delete，显示事务必须先设置自动提交功能为禁用set autocommit=0;

  ```mysql
  开启事务：
  set autocommit=0;start transaction;
  编写sql
  结束事务
  -- 结束时才决定提交事务或回滚事务
  ```

* 并发

  * 脏读：2读取了1没提交的数据（例如1回滚 read uncommitted

  * 不可重复读：2读了数据后1更新 read committed

  * 幻读：2读取了后1插入数据 repeatable read

    ```mysql
    -- 设置当前连接的隔离级别
    set transaction isolation level read committed;
    -- 设置数据库系统的全局的隔离级别
    set global transaction isolation level read committed;
    savepoint 节点名;
    rollback to 节点名;
    ```

    

### 视图

* 视图：虚拟表，和普通表一样使用 重用语句，简化操作，保护数据，提高安全性

* 查看视图

  ```mysql
  desc 视图名；
  show create view 视图名；
  ```

* 视图的更新

  ```
  CREATE OR REPLACE VIEW my_vl 
  AS
  SELECT .....
  或 ALTER VIEW +后续一样
  ```

  视图的增删会影响原表
  tips：包含以下关键字的sql语句：分组函数、distinct、GROUP BY、HAVING 、UNION或者UNION ALL、SELECT、JOIN、FROM、WHERE子句的子查询引用了FROM子句中的视图不允许更新
  
* 视图的删除

  ```mysql
  DROP VIEW 视图名、视图名;
  ```

* 视图和表的区别

  视图创建用create view，基本不占用实际物理空间，只是保存sql逻辑，一般不能增删改，表创建用create table，占用实际物理空间，保存了具体数据

### 变量

* 系统变量

  * 查看所有的系统|会话变量

    ```mysql
    SHOW GLOBAL|SESSION VARIABLES [LIKE '%char%'];
    ```

  * 查看某个指定的系统|会话变量

    ```mysql
    SELECT @@GLOBAL|SESSION.系统变量名
    ```

  * 为某个系统变量赋值 若显式声明则默认为 SESSION			

    ```mysql
    SET GLOBAL|SESSION 名=值;
    SET @@GLOBAL|SESSION.系统变量名=值;
    ```

  * 全局变量：作用域：每次启动全局变量赋初值，对全局会话有效，重启无效						

  * 会话变量：针对于单独的会话（连接）有效

* 自定义变量

### 存储过程和函数

#### 储存过程

> 存储过程是一组预先编译好的sql语句的集合，提高重用性、简化操作，减少编译和连接次数，提高效率

* 创建

  ```
  CREATE PROCEDURE 储存过程名(参数列表)
  BEGIN
  	储存过程体
  END
  ```

  * 参数模式：IN作为输入 OUT作为返回值 INOUT作为输入且返回值
  * 如果存储过程体中仅有一句话则可省略BEGIN END
  * 储存过程中每句sql结尾必须加分号，结尾可以用DELIMITER重新设置：DELIMITER 结束标记

* 调用

  ```mysql
  CALL 储存过程名(实参列表);
  
  DELIMITER $
  CREATE PROCEDURE myp1()
  BEGIN
      INSERT INTO admin(user,`password`)
      VALUES(),(),(),()
  END $
  
  CALL myp1()$
  CREATE PROCEDURE myp2(IN beautyName VARCHAR(20))
  BEGIN
      SELECT bo.*
      FROM boys bo
      RIGHT JOIN beauty b ON bo.id= b.boyfriend_id
      WHERE b.name=beautyName;
  END $
  
  -- 创建储存过程实现传入用户名和密码，插入到admin表中
  CREATE PROCEDURE test_pro1(IN username VARCHAR(20)),IN loginPwd VARCHAR(20)
  BEGIN
      INSERT INTO admin(admin.username,PASSWORD)
      VALUES(username,loginpwd);
  END $
  
  -- 传入女神名称返回 女神and男神格式的字符串
  CREATE PROCEDURE test_pro5(IN beautyName VARCHAR(20),OUT str VARCHAR(50))
  BEGIN
      SELECT CONCAT(beautyName,' and ',IFNULL(boyName,'null')) INTO str
      FROM boys bo							
      RIGHT JOIN beauty b ON b.boyfriend_id=bo.id
      WHERE b.name=beautyName;
  END $
  CALL test_pro5('小昭',@str)$
  SELECT @str $
  ```

* 删除

  ```mysql
  DROP PROCEDURE 储存过程名
  ```

* 查看

  ```mysql
  SHOW CREATE PROCEDURE 存储过程名;
  ```

#### 函数

> 有且只能有一个返回值，多用于处理数据

```mysql
CREATE FUNCTION 函数名(参数列表) RETURNS 返回类型
BEGIN
	函数体
END
CREATE FUNCTION myf1() RETURNS INT
BEGIN
    DECLARE c INT DEFAULT 0;
    SELECT COUNT(*) INTO c
    FROM employees
    RETURNS c;
END $
```

### 流程控制结构

* 顺序结构

* 分支结构

  * IF函数

    ```mysql
    SELECT IF(表达式1,表达式2，表达式3) ->同三目运算符算法
    ```
  
  * CASE
  
    ```mysql
    -- 简单函数：枚举这个字段所有可能的值
    CASE 变量|表达式|字段
    WHEN 要判断的值 THEN 返回的值或语句1;
    WHEN 要判断的值 THEN 返回的值或语句2;
    ...
    ELSE 返回的值或语句3;
    END	CASE;
    -- 搜索函数：可以写判断，并且搜索函数只会返回第一个符合条件的值，其他case被忽略
    CASE 
        WHEN 要判断的表达式 THEN 返回的值或语句1;
        WHEN 要判断的表达式 THEN 返回的值或语句2;
        ...
        ELSE 返回的值或语句3;
    END CASE;
    ```
  
* 循环结构

  ```
  WHILE LOOP REPEAT
  WHILE 循环条件 DO
  	循环体;
  END WHILE[标签];
  -- 例：
  CREATE PROCEDURE pro_while1(IN insertCount INT)
  BEGIN
      DECLAR i INT DEFAULT 1;
      a: WHILE i<=insertCount DO
      INSERT INT amin(username,`passsword`) VALUES(CONCAT('Rose'+i),'666');
      IF i>=20 THEN LEAVE a;
      SET i=i+1;
  END WHILE a;
  END;
  [标签:]LOOP 
  	循环体;
  END LOOP [标签];
  [标签:]REPEAT
  				循环体;
  				UNTIL 结束循环的条件;
  			END REPEAT {标签};
  			循环控制 
  				iterate继续，结束本次循环，继续下一次
  				leave 跳出，结束当前所在的循环
  ```

  